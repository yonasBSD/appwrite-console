<script lang="ts">
    import { Button, InputSearch } from '$lib/elements/forms';
    import { createEventDispatcher } from 'svelte';
    import { sdk } from '$lib/stores/sdk';
    import { Query, type Models } from '@appwrite.io/console';
    import { AvatarInitials, EmptySearch, Modal, PaginationInline } from '..';
    import type { Writable } from 'svelte/store';
    import type { Permission } from './permissions.svelte';
    import {
        Card,
        Empty,
        Layout,
        Link,
        Selector,
        Spinner,
        Table,
        Typography
    } from '@appwrite.io/pink-svelte';
    import { page } from '$app/state';

    export let show: boolean;
    export let groups: Writable<Map<string, Permission>>;

    const dispatch = createEventDispatcher();

    let search = '';
    let offset = 0;
    let isLoading = false;
    let hasSelection = false;
    let selected: Set<string> = new Set();
    let results: Models.TeamList<Record<string, unknown>>;

    function reset() {
        offset = 0;
        search = '';
        selected.clear();
        show = false;
    }

    function create() {
        dispatch('create', Array.from(selected));
        reset();
    }

    async function request() {
        if (!show) return;
        isLoading = true;
        results = await sdk
            .forProject(page.params.region, page.params.project)
            .teams.list([Query.limit(5), Query.offset(offset)], search || undefined);
        isLoading = false;
    }

    function onSelection(role: string) {
        const checked = selected.has(role);
        if (checked) {
            selected.delete(role);
        } else {
            selected.add(role);
        }
        selected = selected;

        hasSelection = selected.size > 0;
    }

    $: if (show) {
        request();
    }
    $: if (offset !== null) {
        request();
    }
    $: if (search !== null) {
        offset = 0;
        request();
    }
</script>

<Modal title="Select teams" bind:show onSubmit={create} on:close={reset}>
    <Typography.Text slot="description"
        >Grant access to any member of a specific team. To grant access to team members with
        specific roles, you will need to set a <Link.Button on:click={() => dispatch('custom')}
            >custom permission</Link.Button
        >.</Typography.Text>
    <InputSearch autofocus placeholder="Search by name or ID" bind:value={search} />
    {#if results?.teams?.length}
        <Table.Root columns={[{ id: 'checkbox', width: 20 }, { id: 'team' }]} let:root>
            {#each results.teams as team (team.$id)}
                {@const role = `team:${team.$id}`}
                {@const exists = $groups.has(role)}
                <Table.Row.Button {root} on:click={() => onSelection(role)} disabled={exists}>
                    <Table.Cell column="checkbox" {root}>
                        <div style:pointer-events="none">
                            <Selector.Checkbox
                                size="s"
                                id={team.$id}
                                disabled={exists}
                                checked={exists || selected.has(role)} />
                        </div>
                    </Table.Cell>
                    <Table.Cell column="team" {root}>
                        <Layout.Stack direction="row" alignItems="center" gap="s">
                            <AvatarInitials size="xs" name={team.name} />
                            <Layout.Stack gap="none">
                                <Typography.Caption variant="400">{team.name}</Typography.Caption>
                                <Typography.Caption
                                    variant="400"
                                    color="--fgcolor-neutral-tertiary">
                                    {team.$id}
                                </Typography.Caption>
                            </Layout.Stack>
                        </Layout.Stack>
                    </Table.Cell>
                </Table.Row.Button>
            {/each}
        </Table.Root>

        <Layout.Stack direction="row" justifyContent="space-between" alignItems="center">
            <p class="text">Total results: {results?.total}</p>
            <PaginationInline
                limit={5}
                bind:offset
                total={results?.total}
                hidePages
                on:change={request} />
        </Layout.Stack>
    {:else if search}
        <EmptySearch bind:search target="teams" hidePages>
            <Button
                external
                href="https://appwrite.io/docs/products/auth/teams"
                text
                event="empty_documentation"
                size="s">Documentation</Button>
            <Button secondary on:click={() => (search = '')}>Clear search</Button>
        </EmptySearch>
    {:else if isLoading}
        <!-- 275px nearly matches the height of at-least 5 items in the table above -->
        <div style:margin-inline="auto" style:min-height="275px" style:align-content="center">
            <Spinner size="m" />
        </div>
    {:else}
        <Card.Base padding="none">
            <Empty title="You have no teams. Create a team to see them here." type="secondary">
                <Typography.Text slot="description">
                    Need a hand? Learn more in our <Link.Anchor
                        href="https://appwrite.io/docs/products/auth/teams"
                        target="_blank"
                        rel="noopener noreferrer">
                        documentation</Link.Anchor
                    >.
                </Typography.Text>
            </Empty>
        </Card.Base>
    {/if}

    <svelte:fragment slot="footer">
        <Button submit disabled={!hasSelection}>Add</Button>
    </svelte:fragment>
</Modal>
