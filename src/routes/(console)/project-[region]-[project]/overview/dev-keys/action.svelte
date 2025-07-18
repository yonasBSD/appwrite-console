<script lang="ts">
    import Button from '$lib/elements/forms/button.svelte';
    import { canWriteKeys } from '$lib/stores/roles';
    import { Icon, Modal, Layout } from '@appwrite.io/pink-svelte';
    import { IconPlus } from '@appwrite.io/pink-icons-svelte';
    import { Form, InputText } from '$lib/elements/forms/index.js';
    import { ExpirationInput } from '$lib/components';
    import { sdk } from '$lib/stores/sdk';
    import { goto } from '$app/navigation';
    import { Submit, trackError, trackEvent } from '$lib/actions/analytics';
    import { base } from '$app/paths';
    import { addNotification } from '$lib/stores/notifications';
    import { page } from '$app/state';
    import { showDevKeysCreateModal } from '$routes/(console)/project-[region]-[project]/overview/store';

    let name = '';
    let expire = null;
    let isSubmitting = false;
    const projectId = page.params.project;

    async function create() {
        try {
            isSubmitting = true;
            const { $id } = await sdk.forConsole.projects.createDevKey(projectId, name, expire);

            $showDevKeysCreateModal = false;
            trackEvent(Submit.DevKeyCreate);
            await goto(
                `${base}/project-${page.params.region}-${page.params.project}/overview/dev-keys/${$id}`
            );
            addNotification({
                message: `Dev key has been created`,
                type: 'success'
            });
        } catch (error) {
            addNotification({
                type: 'error',
                message: error.message
            });
            trackError(error, Submit.DevKeyCreate);
        } finally {
            isSubmitting = false;
        }
    }
</script>

{#if $canWriteKeys}
    <Button on:click={() => ($showDevKeysCreateModal = true)}>
        <Icon icon={IconPlus} slot="start" size="s" />
        Create dev key
    </Button>

    <Form onSubmit={create} isModal>
        <Modal title="Create dev key" bind:open={$showDevKeysCreateModal}>
            <span slot="description">
                Bypass Appwrite rate limits and CORS errors in your development environment.
            </span>

            <Layout.Stack>
                <InputText
                    id="name"
                    label="Name"
                    placeholder="Enter key name"
                    required
                    bind:value={name} />

                <ExpirationInput bind:value={expire} expiryOptions="limited" />
            </Layout.Stack>

            <Layout.Stack direction="row" justifyContent="flex-end" slot="footer">
                <Button fullWidthMobile secondary on:click={() => ($showDevKeysCreateModal = false)}
                    >Cancel</Button>
                <Button fullWidthMobile submit disabled={isSubmitting}>Create</Button>
            </Layout.Stack>
        </Modal>
    </Form>
{/if}
