<script lang="ts">
    let { list = false, items, value = $bindable() } = $props();
</script>

{#if list}
    <select bind:value>
        {#each items as item}
            <option value={item.value}>{item.name}</option>
        {/each}
    </select>
{:else}
    <div class="dropdown">
        {#each items as item}
            <button type="button" class:selected={value == item.value} onclick={() => (value = item.value)}
                >{item.name}</button
            >
        {/each}
    </div>
{/if}

<style lang="scss">
    .dropdown {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;
    }
    .dropdown button {
        background-color: #ddd;
        padding: 0.25rem;
        margin: 0.1rem;
        white-space: nowrap;
    }
    .dropdown button.selected {
        background-color: rgb(173, 251, 173);
    }
</style>
