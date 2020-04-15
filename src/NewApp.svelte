<script>

    import { get_solution } from './model';

    const countries = [{
        id: 'uk',
        displayName: 'United Kingdom'
    }];

    let activeCountry = countries.find(c => c.id === 'uk');

    const interventions = [{
        id: 'do-nothing',
        displayName: 'Do nothing',
        transmissionRateChange: 0.0,
        active: false
    }, {
        id: 'stay-at-home',
        displayName: 'Stay at home',
        transmissionRateChange: -0.5,
        active: false
    }, {
        id: 'stay-at-home-plus-contact-tracing',
        displayName: 'Stay at home + contact tracing',
        transmissionRateChange: -1.0,
        active: false
    }];

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

    var solution = get_solution(
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
        InterventionAmt,
        duration);

    console.log(solution);
</script>

<style>

    /* Chart section */

    /* Chart section end */

    /* Estimated outcomes section */

    td.r-below-one {
        color: #0a0;
        font-weight: bold;
    }

    /* Estimated outcomes section end */

</style>

<header>
    Logo on the left, other top-level navigation elements on the right
    <nav>
        <ul>
            <li>Nav 1</li>
            <li>Nav 2</li>
            <li>Nav 3</li>
        </ul>
    </nav>
</header>

<main>
    <section>
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

        <div>Transmission rate info box</div>
        <div>Current intervention info box</div>

        <div>
            <!--
                NOTE: these checkboxes determine which curves are shown on the chart
            -->
            {#each interventions as intervention}
                <label>
                    <input
                        type="checkbox"
                        bind:checked={intervention.active} />
                    {intervention.displayName}
                </label>
            {/each}
        </div>
        <div id="chart">
            <!-- chart lives here! -->
        </div>
    </section>

    <section>
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
    </section>
</main>