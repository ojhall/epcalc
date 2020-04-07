<script>
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
        <h1>United Kingdom</h1>
        <select id="location">
            <option value="uk">United Kingdom</option>
        </select>
        <div>Transmission rate box</div>
        <div>Current intervention box</div>
        <div>
            Chart goes here!
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
                <td>Stay at home + Contact tracing</td>
                <td class="r-below-one">0.1 - 0.5</td>
                <td>220,498</td>
                <td>-</td>
                <td>12,000</td>
            </tr>
        </table>
    </section>
</main>