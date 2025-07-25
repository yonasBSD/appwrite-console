<script lang="ts">
    import { Click, Submit, trackEvent } from '$lib/actions/analytics';
    import { CardGrid } from '$lib/components';
    import { Alert } from '@appwrite.io/pink-svelte';
    import { BillingPlan } from '$lib/constants';
    import { Button, Form, InputNumber, InputSelect } from '$lib/elements/forms';
    import { humanFileSize, sizeToBytes } from '$lib/helpers/sizeConvertion';
    import { createByteUnitPair } from '$lib/helpers/unit';
    import { readOnly, upgradeURL } from '$lib/stores/billing';
    import { organization } from '$lib/stores/organization';
    import { GRACE_PERIOD_OVERRIDE, isCloud } from '$lib/system';
    import { updateBucket } from './+page.svelte';
    import type { Plan } from '$lib/sdk/billing';
    import type { Models } from '@appwrite.io/console';

    export let bucket: Models.Bucket;
    export let currentPlan: Plan | null;

    const service = currentPlan ? currentPlan['fileSize'] : null;
    const { value, unit, baseValue, units } = createByteUnitPair(bucket.maximumFileSize, 1000);
    const options = units.map((v) => ({ label: v.name, value: v.name }));
    $: selectedUnit = $unit;

    const maxValue = function formMaxFileSize() {
        return (service * 1000 * 1000) / units.find((unit) => unit.name === selectedUnit).value;
    };

    function updateMaxSize() {
        updateBucket(
            bucket,
            {
                maximumFileSize: $baseValue
            },
            {
                trackEventName: Submit.BucketUpdateSize,
                arePermsDisabled: false
            }
        );
    }
</script>

<Form onSubmit={updateMaxSize}>
    <CardGrid>
        <svelte:fragment slot="title">Maximum file size</svelte:fragment>
        Set the maximum file size allowed in the bucket.
        <svelte:fragment slot="aside">
            {#if isCloud}
                {@const size = humanFileSize(sizeToBytes(service, 'MB', 1000))}
                <Alert.Inline status="info">
                    The {currentPlan.name} plan has a maximum upload file size limit of {Math.floor(
                        parseInt(size.value)
                    )}{size.unit}.
                    {#if $organization?.billingPlan === BillingPlan.FREE}
                        Upgrade to allow files of a larger size.
                    {/if}
                    <svelte:fragment slot="actions">
                        {#if $organization?.billingPlan === BillingPlan.FREE}
                            <div class="alert-buttons u-flex">
                                <Button
                                    text
                                    href={$upgradeURL}
                                    on:click={() => {
                                        trackEvent(Click.OrganizationClickUpgrade, {
                                            source: 'bucket_settings'
                                        });
                                    }}>Upgrade plan</Button>
                            </div>
                        {/if}
                    </svelte:fragment>
                </Alert.Inline>
            {/if}
            <InputNumber
                required
                id="size"
                label="Size"
                disabled={$readOnly && !GRACE_PERIOD_OVERRIDE}
                placeholder={bucket.maximumFileSize.toString()}
                min={0}
                max={isCloud ? maxValue() : Infinity}
                bind:value={$value} />
            <InputSelect required id="bytes" label="Bytes" {options} bind:value={$unit} />
        </svelte:fragment>

        <svelte:fragment slot="actions">
            <Button
                disabled={$baseValue === bucket.maximumFileSize ||
                    ($readOnly && !GRACE_PERIOD_OVERRIDE)}
                submit>
                Update
            </Button>
        </svelte:fragment>
    </CardGrid>
</Form>
