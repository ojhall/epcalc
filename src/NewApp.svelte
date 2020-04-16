<script>

    import * as d3 from "d3";
    import { get_solution } from './model';

    const countries = [{
        id: 'uk',
        displayName: 'United Kingdom'
    }];

    let activeCountry = countries.find(c => c.id === 'uk');

    let interventions = [{
        id: 'do-nothing',
        displayName: 'Do nothing',
        transmissionRateFactor: 1.0,
        active: false
    }, {
        id: 'stay-at-home',
        displayName: 'Stay at home',
        transmissionRateFactor: 0.6,
        active: false
    }, {
        id: 'stay-at-home-plus-contact-tracing',
        displayName: 'Stay at home + contact tracing',
        transmissionRateFactor: 0.4,
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
    var logN              = Math.log(7e6)
    var N                 = Math.exp(logN)
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
    var OMInterventionAmt = 2/3
    var InterventionAmt   = 1 - OMInterventionAmt
    var dt                = 2
    var P_SEVERE          = 0.2
    var duration          = 7*12*1e10

    $: activeInterventions = interventions.filter(i => i.active);
    $: solutions = activeInterventions.map(i => getSolutionForIntervention(i.transmissionRateFactor));

    const getSolutionForIntervention = (transmissionRateFactor) => {
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
            InterventionAmt * transmissionRateFactor,
            duration);
    }

    // chart stuff!

    $: solutionData = solutions.map(solution => solution.P.map(d => d[0]));

    var xMax = 0;
    var yMax = 0;
    var solutionData = [];
    $: solutionData.forEach(solutionData => {
        xMax = Math.max(xMax, solutionData.length);
        yMax = Math.max(yMax, solutionData.reduce((a, b) => a > b ? a : b));
    });

    const graphWidth = 800;
    const graphHeight = 400;

    $: xScale = d3.scaleLinear()
        .domain([0, xMax])
        .range([0, graphWidth]);
    $: yScale = d3.scaleLinear()
        .domain([0, yMax])
        .range([graphHeight, 0]); // flipped around for SVG co-ord system
    $: lineFunction = d3.line()
        .x((d, i) => xScale(i))
        .y(d => yScale(d))
        .curve(d3.curveMonotoneX);
</script>

<style>

    * {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
            <h1>{activeCountry.displayName}</h1>
            <select bind:value={activeCountry}>
                <!--
                    NOTE: setting this value may update things like:
                    * historical data
                    * population size
                    * enacted interventions
                -->
                {#each countries as country}
                    <option value={country}>{country.displayName}</option>
                {/each}
            </select>

            <div>
                <div class="info-box">Transmission rate info box</div>
                <div class="info-box">Current intervention info box</div>
            </div>

            <div class="no-highlighting">
                <!--
                    NOTE: these checkboxes determine which curves are shown on the chart
                -->
                {#each interventions as intervention}
                    <label>
                        <input
                            type="checkbox"
                            checked={intervention.active}
                            on:change={() => setInterventionActive(intervention.id, !intervention.active)} />
                        {intervention.displayName}
                    </label>
                {/each}
            </div>
            <div id="chart">
                <svg width={graphWidth} height={graphHeight}>
                    <g>
                        {#each solutionData as solutionDatum}
                            <path
                                fill="none"
                                stroke="steelblue"
                                stroke-width="1.5"
                                d={lineFunction(solutionDatum)}
                            >
                            </path>
                        {/each}
                    </g>
                </svg>
            </div>
        </div>
    </section>

    <section>
        <div class="section-content">
            <h2>Estimated outcomes</h2>
            <h3>22 Mar - 22 Jun 2020</h3>
            <p>
                Bit of an overview to caveat the figures in the table below and to give a bit of context around how these figures can be calculated.
                Lorem ipsum dolor sit ame tconsectetur adipiscing elit. In in pellentesque risus, tristique placerat massa.
                <a href="https://www.example.com" target="_blank" rel="noopener noreferrer">Learn more</a>
            </p> 
            <table>
                <tr>
                    <th>Intervention</th>
                    <th>Transmission rate min-max range</th>
                    <th>Total hospitalised</th>
                    <th>Date hospital capacity reached</th>
                    <th>Total fatalities</th>
                </tr>
                <tr>
                    <td>Do nothing</td>
                    <td>0.5 - 1.5</td>
                    <td>350,456</td>
                    <td>22 Apr 2020</td>
                    <td>22,000</td>
                </tr>
                <tr>
                    <td>Stay at home (current)</td>
                    <td class="r-below-one">0.2 - 0.7</td>
                    <td>298,132</td>
                    <td>2 May 2020</td>
                    <td>19,898</td>
                </tr>
                <tr>
                    <td>Stay at home + contact tracing</td>
                    <td class="r-below-one">0.1 - 0.5</td>
                    <td>220,498</td>
                    <td>-</td>
                    <td>12,000</td>
                </tr>
            </table>
        </div>
    </section>
</main>