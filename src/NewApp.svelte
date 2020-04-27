<script>

    import * as d3 from "d3";
    import NewCheckbox from './NewCheckbox.svelte';
    import { get_solution } from './model';
    import countries from './population-data-2018';

    function insertCommas(num) {
        return num.toString().replace(/(\d)(?=(\d{3})+(?!\d))/g, '$1,');
    }

    function formatNumberForAxis(num) {
        if (num < 1000) {
            return num;
        }
        if (num < 1000000) {
            return (num/1000) + 'k';
        }
        return (num/1000000) + 'M';
    }

    let activeCountry = countries.find(c => c.name === 'United Kingdom');

    let interventions = [{
        id: 'do-nothing',
        displayName: 'Do nothing',
        transmissionRateFactor: 1.0,
        color: '#ED388D',
        active: true
    }, {
        id: 'stay-at-home',
        displayName: 'Stay at home',
        transmissionRateFactor: 0.6,
        color: '#336DC2',
        active: false
    }, {
        id: 'stay-at-home-plus-contact-tracing',
        displayName: 'Stay at home + contact tracing',
        transmissionRateFactor: 0.4,
        color: '#1AAC1E',
        active: false
    }];

    const setInterventionActive = (id, active) => {
        const intervention = interventions.find(i => i.id === id);
        intervention.active = active;
        interventions = interventions; // make Svelte update itself - can be done better
    }

    var historicalData = [];
    var modelledData = [];

    var Time_to_death     = 32
    $: N                  = activeCountry.population;
    var I0                = 1
    var R0                = 2.2
    var D_incbation       = 5.2       
    var D_infectious      = 2.9 
    var D_recovery_mild   = (14 - 2.9)  
    var D_recovery_severe = (31.5 - 2.9)
    var D_hospital_lag    = 5
    var D_death           = Time_to_death - D_infectious 
    var CFR               = 0.02  
    var InterventionTime  = 100
    var dt                = 4
    var P_SEVERE          = 0.2
    var duration          = 7*12*1e10

    $: interventionResults = interventions.map(i => {
        const solution = getSolutionForIntervention(i.transmissionRateFactor);
        const deaths = solution.P.map(d => d[0]);
        const hospitalised = solution.P.map(d => d[1]);
        return {
            ...i,
            hospitalised: hospitalised,
            maxHospitalised: Math.ceil(hospitalised.reduce((total, i) => Math.max(total, i), 0)),
            maxDeaths: Math.ceil(deaths.reduce((total, i) => Math.max(total, i), 0)),
        };
    });

    $: getSolutionForIntervention = (transmissionRateFactor) => {
        return get_solution(
            dt,
            N,
            I0,
            R0,
            D_incbation,
            D_infectious,
            D_recovery_mild,
            D_hospital_lag,
            D_recovery_severe,
            D_death,
            P_SEVERE,
            CFR,
            InterventionTime,
            transmissionRateFactor,
            duration);
    }

    // chart stuff!

    const graphWidth = 800;
    const graphHeight = 400;
    const padding = { top: 20, right: 0, bottom: 20, left: 25 };

    $: xMax = interventionResults.map(i => i.hospitalised.length).reduce((total, num) => Math.max(total, num), 0);
    $: yMax = interventionResults.reduce((total, i) => Math.max(total, i.maxHospitalised), 0);

    $: xScale = d3.scaleLinear()
        .domain([0, xMax])
        .range([padding.left, graphWidth - padding.right]);
    $: yScale = d3.scaleLinear()
        .domain([0, yMax])
        .range([graphHeight - padding.bottom, padding.top]); // flipped around for SVG co-ord system
    $: lineFunction = d3.line()
        .x((d, i) => xScale(i))
        .y(d => yScale(d))
        .curve(d3.curveMonotoneX);
    $: areaFunction = d3.area()
        .x((d, i) => xScale(i))
        .y0(graphHeight - padding.bottom)
        .y1(d => yScale(d));
</script>

<style>

    * {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    .unimplemented {
        color: lightgray;
    }

    header {
        max-width: 1000px;
        margin: auto;
    }

    header > span {
        float: left;
    }

    nav > ul > li {
        display: inline-block;
    }

    main {
        max-width: 1000px;
        margin: auto;
    }

    .section-content {
        max-width: 800px;
        margin: auto;
    }

    /* Chart section */

    section#chart-section {
        background: #f1f5f7;
        border-top: 2px solid #dee2e4;
        padding-top: 1em;
        padding-bottom: 1em;
    }

    select {
        padding: 10px;
        margin: 10px 10px 10px 0px;
        border-radius: 5px;
        border: 1px solid #dee2e4;
    }

    select::-ms-expand {
	    display: none;
    }

    .info-box {
        display: inline-block;
        background: #fff;
        padding: 10px;
        margin: 10px 10px 10px 0px;
        border-radius: 5px;
        -moz-appearance: none;
	    -webkit-appearance: none;
	    appearance: none;
    }

    .no-highlighting {
        user-select: none; /* supported by Chrome and Opera */
        -webkit-user-select: none; /* Safari */
        -khtml-user-select: none; /* Konqueror HTML */
        -moz-user-select: none; /* Firefox */
        -ms-user-select: none; /* Internet Explorer/Edge */
    }

    .intervention-checkbox-container {
        display: flex;
        flex-wrap: nowrap;
        justify-content: flex-start;
        align-items: center;
    }

    .intervention-checkbox {
        font-weight: bold;
        margin-right: 25px;
    }

    /* Chart section end */

    /* Estimated outcomes section */

    td.r-below-one {
        color: #0a0;
        font-weight: bold;
    }

    /* Estimated outcomes section end */

</style>

<header>
    <span>
        Logo on the left, other top-level navigation elements on the right
    </span>
    <nav>
        <ul>
            <li>Nav 1</li>
            <li>Nav 2</li>
            <li>Nav 3</li>
        </ul>
    </nav>
</header>

<main>
    <section id="chart-section">
        <div class="section-content">
            <h1>{activeCountry.name}</h1>
            <select bind:value={activeCountry}>
                <!--
                    NOTE: setting this value may update things like:
                    * historical data
                    * population size
                    * enacted interventions
                -->
                {#each countries as country}
                    <option value={country}>{country.name}</option>
                {/each}
            </select>

            <div>
                <div class="info-box">Transmission rate info box</div>
                <div class="info-box">Current intervention info box</div>
            </div>

            <div class="intervention-checkbox-container no-highlighting">
                <!--
                    NOTE: these checkboxes determine which curves are shown on the chart
                -->
                {#each interventions as intervention}
                    <span class="intervention-checkbox">
                        <NewCheckbox
                            label={intervention.displayName}
                            color={intervention.color}
                            checked={intervention.active}
                            callback={checked => setInterventionActive(intervention.id, checked)} />
                    </span>
                {/each}
            </div>
            <div id="chart">
                <svg width={graphWidth} height={graphHeight}>
                    <g class="axis y-axis" transform="translate(0,{padding.top})">
                        {#each yScale.ticks(5) as tick}
                            <g class="tick tick-{tick}" transform="translate(0, {yScale(tick) - padding.bottom})">
                            <line x2="100%"></line>
                            <text y="-4">{formatNumberForAxis(tick)}{ (tick == yScale.ticks(5)[0]) ? " ": ""}</text>
                            </g>
                        {/each}
                    </g>

                    <!-- x axis -->
                    <g class="axis x-axis">
                        {#each xScale.ticks() as i}
                            <g class="tick" transform="translate({xScale(i)},{graphHeight})">
                            <text x="0" y="-4">{i == 0 ? "Day ":""}{i}</text>
                            </g>
                        {/each}
                    </g>

                    <g>
                        {#each interventionResults as interventionResult}
                            {#if interventionResult.active}
                                <path
                                    stroke={interventionResult.color}
                                    stroke-width="1.5"
                                    fill="none"
                                    d={lineFunction(interventionResult.hospitalised)}>
                                </path>
                                <path
                                    stroke-width="0"
                                    fill={interventionResult.color}
                                    opacity="0.2"
                                    d={areaFunction(interventionResult.hospitalised)}>
                                </path>
                            {/if}
                        {/each}
                    </g>
                </svg>
            </div>
        </div>
    </section>

    <section>
        <div class="section-content">
            <h2>Estimated outcomes</h2>
            <h3 class="unimplemented">22 Mar - 22 Jun 2020</h3>
            <p>
                Bit of an overview to caveat the figures in the table below and to give a bit of context around how these figures can be calculated.
                Lorem ipsum dolor sit ame tconsectetur adipiscing elit. In in pellentesque risus, tristique placerat massa.
                <a href="https://www.example.com" target="_blank" rel="noopener noreferrer">Learn more</a>
            </p> 
            <table>
                <tr>
                    <th>Intervention</th>
                    <th>Estimated transmission rate</th>
                    <th>Total hospitalised</th>
                    <th>Date hospital capacity reached</th>
                    <th>Total fatalities</th>
                </tr>
                {#each interventionResults as interventionResult}
                    <tr>
                        <td>{interventionResult.displayName}</td>
                        <td>{(R0 * interventionResult.transmissionRateFactor).toFixed(2)}</td>
                        <td>{insertCommas(interventionResult.maxHospitalised)}</td>
                        <td class="unimplemented">22 Apr 2020</td>
                        <td>{insertCommas(interventionResult.maxDeaths)}</td>
                    </tr>
                {/each}
            </table>
        </div>
    </section>
</main>