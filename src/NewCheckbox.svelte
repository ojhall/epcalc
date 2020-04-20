<script>
    export let label;
    export let color;
    export let checked = false;
    export let callback = () => {};
</script>

<style>
    .tick {
        cursor: default;
        font-size: 12px;
        color: white;
        text-align: center;
        vertical-align: middle;
        line-height: 13px;
    }
</style>

<label on:click={() => {checked = !checked; callback(checked)}}>
    <div
        style="display: inline-block;
            width: 13px;
            height: 13px;
            background-color: {checked ? color:'white'};
            border-radius: 3px;
            opacity: 0.8;
            border: 2px solid {checked ? 'rgba(255,255,255,0)':color };
            box-shadow: inset 1px 1px 3px rgba(0, 0, 0, 0.1);
            user-select: none;"
        >
        <div class="tick">
            {checked ? "✓" : "."}
        </div>
    </div>
    {label + ' '}
</label>
