<script>

    import { onMount } from 'svelte'

    export let min = 0
    export let max = 9
    export let value = min
    export let itemWidth = 0.2
    export let gapWidth = 0.05

    let options = [];
    let wrapRef = null
    let ulRef = null;
    for (let i = min; i <= max; i++) options.push(i);

    // state used for dragging
    let dragging = false
    let dragStartX = 0
    let translateX = 0
    let translateOldX = 0
    let ignoreClick = false

    // sizes used for calculations
    let total_px
    let item_px
    let gap_px
    let midpoint_px

    $: !dragging && value===0 && ulRef && ( calcSizes(), snap(0) )

    function calcSizes () {
        total_px    = wrapRef.getBoundingClientRect().width
        item_px     = total_px * itemWidth
        gap_px      = total_px * gapWidth
        midpoint_px = total_px * 0.5
    }

    function calcSelected () {
        return Math.floor((midpoint_px - translateX) / (item_px + gap_px))
    }

    function translate (x) {
        let prop = `translate(${ x.toString() }px)`
        ulRef.style.setProperty('transform', prop)
    }

    function snap(ndx) {
        translateX = midpoint_px - (ndx + 0.5)*(item_px + gap_px)
        translate(translateX)
        ulRef.style.setProperty('transition', '0.2s ease')
    }

    // dont allow user to scroll past bounds
    function bound(x) {
        const right = midpoint_px
        const left = midpoint_px - (item_px + gap_px) * options.length + 1
        if (x > right) return right
        else if (x < left) return left
        else return x
    }

    function dragStart(event) {
        dragging = true;
        dragStartX = event.clientX || event.touches[0].clientX;
        translateOldX = translateX
        ulRef.style.setProperty('transition', 'none')
        calcSizes()
    }

    function dragSlide(event) {
        event.preventDefault()
        if (!dragging) return
        const clientX = event.clientX || event.touches[0].clientX
        let delta = clientX - dragStartX;
        translateX = bound(translateOldX + delta)
        translate(translateX)
        let ndx = calcSelected()
        value = options[ndx]
    }

    function dragStop(event) {
        if (!dragging) return
        dragging = false
        const ndx = calcSelected()
        if (Math.abs(translateX - translateOldX) > 10) ignoreClick = true
        snap(ndx)
    }

    function increment () {
        calcSizes();
        const ndx = calcSelected() + 1
        value = options[ndx]
        snap(ndx)
    }

    function decrement () {
        calcSizes()
        const ndx = calcSelected() - 1
        value = options[ndx]
        snap(ndx)
    }

    function select(ndx) {
        if (ignoreClick) {
            ignoreClick = false
        }
        else {
            value = options[ndx]
            snap(ndx)
        }
    }

    onMount(() => {
        calcSizes()
        snap(0)
    })

</script>

<style>

    .container {
        display: flex;
        font-size: 1.5rem;
        color: var(--grey);
        position: relative;
    }

    .line {
        position: absolute;
        left: calc(50% - 30px);
        top: 12px;
        width: 60px;
        height: 48px;
        border-radius: 1em;
        border: 1px solid white;
    }

    .arrow {
        width: 20%;
        line-height: 1.5rem;
        color: var(--grey);
    }

    .arrow:hover {
        cursor: pointer;
    }

    .wrap {
        overflow: hidden;
        border: none;
        flex-grow: 1;
        padding: 0em 0;
    }

    ul {
        list-style: none;
        padding: 0;
        margin: 0;
        white-space: nowrap;
        overflow: visible;
        height: 100%;
    }

    li {
        display: inline-block;
        vertical-align: middle;
        width: var(--item-width);
        margin: 0.5em calc(0.5 * var(--gap-width));
        text-align: center;
        border-radius: 3em;
        padding: 0.5em 0;
    }

    li.active {
        color: var(--green-dark);
        background-color: white;
    }

    button {
        width: 100%;
        margin: 0;
        text-align: center;
        font-size: inherit;
        font-family: inherit;
        border: none;
        background: transparent;
    }

    button:focus {
        outline: none;
    }

</style>

<div class="container" style="--item-width: {itemWidth*100}%; --gap-width: {gapWidth*100}%">

    <button
        class="left arrow"
        on:click={decrement}
        disabled={value===min}
    >&#60;</button>

    <div
        class="wrap"
        bind:this={wrapRef}
    >
        <ul bind:this={ulRef}>
            {#each options as option, i}
                <li class:active={option===value} on:click={() => select(i)}>
                    {option}
                </li>
            {/each}
        </ul>
    </div>

    <button
        class="right arrow"
        on:click={increment}
        disabled={value===max}
    >&#62;</button>

</div>