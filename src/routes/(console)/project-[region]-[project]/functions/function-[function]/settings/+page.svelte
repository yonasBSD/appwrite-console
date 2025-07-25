<script lang="ts">
    import { Container } from '$lib/layout';
    import DangerZone from './dangerZone.svelte';
    import ExecuteFunction from './executeFunction.svelte';
    import UpdateEvents from './updateEvents.svelte';
    import UpdateLogging from './updateLogging.svelte';
    import UpdateName from './updateName.svelte';
    import UpdatePermissions from './updatePermissions.svelte';
    import UpdateRuntime from './updateRuntime.svelte';
    import UpdateSchedule from './updateSchedule.svelte';
    import UpdateScopes from './updateScopes.svelte';
    import UpdateTimeout from './updateTimeout.svelte';
    import { sdk } from '$lib/stores/sdk';
    import { Dependencies } from '$lib/constants';
    import { invalidate } from '$app/navigation';
    import { Alert } from '@appwrite.io/pink-svelte';
    import { Button } from '$lib/elements/forms';
    import { Click, trackEvent } from '$lib/actions/analytics';
    import UpdateRepository from './updateRepository.svelte';
    import UpdateBuildCommand from './updateBuildCommand.svelte';
    import UpdateResourceLimits from './updateResourceLimits.svelte';
    import { isCloud } from '$lib/system';
    import UpdateVariables from '$routes/(console)/project-[region]-[project]/updateVariables.svelte';
    import { page } from '$app/state';
    import { Link } from '$lib/elements';

    export let data;
    let showAlert = true;

    const sdkCreateVariable = async (key: string, value: string, secret?: boolean) => {
        await sdk
            .forProject(page.params.region, page.params.project)
            .functions.createVariable(data.function.$id, key, value, secret);
        await Promise.all([invalidate(Dependencies.VARIABLES), invalidate(Dependencies.FUNCTION)]);
    };

    const sdkUpdateVariable = async (
        variableId: string,
        key: string,
        value: string,
        secret?: boolean
    ) => {
        await sdk
            .forProject(page.params.region, page.params.project)
            .functions.updateVariable(data.function.$id, variableId, key, value, secret);
        await Promise.all([invalidate(Dependencies.VARIABLES), invalidate(Dependencies.FUNCTION)]);
    };

    const sdkDeleteVariable = async (variableId: string) => {
        await sdk
            .forProject(page.params.region, page.params.project)
            .functions.deleteVariable(data.function.$id, variableId);
        await Promise.all([invalidate(Dependencies.VARIABLES), invalidate(Dependencies.FUNCTION)]);
    };
</script>

<Container>
    {#if data.function.version === 'v2' && showAlert}
        <div>
            <Alert.Inline
                status="warning"
                dismissible
                title="Your function is outdated"
                on:dismiss={() => (showAlert = false)}>
                Update your function version to make use of new features including build commands
                and HTTP data in your executions. To update, follow the steps outlined in our
                <Link href="https://appwrite.io/docs/products/functions/development" external
                    >documentation</Link
                >.
                <svelte:fragment slot="actions">
                    <Button
                        on:click={() =>
                            trackEvent(Click.WebsiteOpenClick, {
                                from: 'button',
                                source: 'function_keys_card',
                                destination: 'docs'
                            })}
                        href="https://appwrite.io/docs/products/functions/development"
                        external
                        text>
                        Learn more
                    </Button>
                </svelte:fragment>
            </Alert.Inline>
        </div>
    {/if}
    <ExecuteFunction />
    <UpdateName />
    <UpdateRuntime runtimesList={data.runtimesList} />
    {#key data.function.providerRepositoryId}
        <UpdateRepository func={data.function} installations={data.installations} />
    {/key}

    <UpdateVariables
        {sdkCreateVariable}
        {sdkUpdateVariable}
        {sdkDeleteVariable}
        isGlobal={false}
        globalVariableList={data.globalVariables}
        variableList={data.variables}
        analyticsSource="function_settings" />
    <UpdateBuildCommand func={data.function} />

    <UpdatePermissions />
    {#if isCloud}
        <UpdateResourceLimits func={data.function} specs={data.specificationsList} />
    {/if}
    <UpdateEvents />
    <UpdateSchedule />
    <UpdateTimeout />
    <UpdateLogging func={data.function} />
    <UpdateScopes />
    <DangerZone />
</Container>
