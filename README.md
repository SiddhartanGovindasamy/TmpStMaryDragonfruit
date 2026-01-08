<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>ThingSpeak – Filtered by Time</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 30px;
        }
        .chart-container {
            max-width: 1000px;
            margin-bottom: 50px;
        }
    </style>
</head>
<body>

<h2>ThingSpeak – Data After Jan 7, 05:30</h2>

<div class="chart-container">
    <canvas id="tempChart"></canvas>
</div>

<div class="chart-container">
    <canvas id="batteryChart"></canvas>
</div>

<script>
/* ================= SETTINGS ================= */
const CHANNELS = [
    { id: 3028067, label: "3028067" },
    { id: 3028069, label: "3028069" }
];
const RESULTS = 100;

/* ---- TIME FILTER (UTC) ---- */
const CUTOFF_TIME = new Date(2026, 0, 7, 5, 40, 0, 0).getTime();

/* =========================================== */

async function fetchFeeds(channelId) {
    const url = `https://api.thingspeak.com/channels/${channelId}/feeds.json?results=${RESULTS}`;
    const res = await fetch(url);
    const json = await res.json();
    return json.feeds || [];
}

function datasetFromFeeds(feeds, field, label) {
    return {
        label,
        data: feeds
            .filter(f =>
                f[field] !== null &&
                new Date(f.created_at).getTime() >= CUTOFF_TIME
            )
            .map(f => ({
                x: new Date(f.created_at).getTime(),
                y: Number(f[field])
            })),
        borderWidth: 2,
        tension: 0.2
    };
}

function buildChart(canvasId, title, field, yLabel) {
    Promise.all(CHANNELS.map(c => fetchFeeds(c.id))).then(feeds => {

        new Chart(document.getElementById(canvasId), {
            type: "line",
            data: {
                datasets: [
                    datasetFromFeeds(feeds[0], field, CHANNELS[0].label),
                    datasetFromFeeds(feeds[1], field, CHANNELS[1].label)
                ]
            },
            options: {
                responsive: true,
                parsing: false,
                plugins: {
                    title: {
                        display: true,
                        text: title
                    },
                    tooltip: {
                        callbacks: {
                            title: items =>
                                new Date(items[0].parsed.x).toLocaleString()
                        }
                    }
                },
                scales: {
                    x: {
                        type: "linear",
                        title: {
                            display: true,
                            text: "Timestamp"
                        },
                        ticks: {
                            callback: v =>
                                new Date(v).toLocaleTimeString()
                        }
                    },
                    y: {
                        title: {
                            display: true,
                            text: yLabel
                        }
                    }
                }
            }
        });

    });
}

/* -------- Draw charts -------- */
buildChart(
    "tempChart",
    "Temperature (Field 5) – After Jan 7, 05:30",
    "field5",
    "Temperature"
);

buildChart(
    "batteryChart",
    "Battery Voltage (Field 7) – After Jan 7, 05:30",
    "field7",
    "Voltage (V)"
);
</script>

</body>
</html>

