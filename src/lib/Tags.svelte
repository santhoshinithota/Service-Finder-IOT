<script>
    export let serviceList;
    const tags = [ 'Bathroom', 'Other Services' ]
    const tagId = {
        'Bathroom': 'bathroom',
        'Other Services': 'other'
    }
</script>

<div class="tags columns">
{#each tags as tag}
    <span class="tag column col-auto chip text-large bg-dark text-light">
        <button on:click={() => serviceList.Display(tagId[tag])} class="btn btn-link text-light">{tag}</button>
    </span>
{/each}
</div>

<style>
.tags
{
    height: 4%;
    width: 100%;

    align-items: center;
    padding: 1% 2%;
}

.tag
{
    z-index: 2;
    height: 100%;
}

.btn
{
    border: none;
    background: none;
    color: lightgrey;
}
</style>
