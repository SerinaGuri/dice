
<script>
    import { onMount } from 'svelte';

    let number = $state(null);
    let status = $state('Waiting for motion permission...');

    let gx = 0, gy = 0, gz = 0;
    let lastShake = 0;

    function roll() {
        const values = [
            { axis: 'x', value: gx },
            { axis: 'y', value: gy },
            { axis: 'z', value: gz }
        ].sort((a, b) => Math.abs(b.value) - Math.abs(a.value));

        const { axis, value } = values[0];

        if (axis === 'z') number = value < 0 ? 1 : 6;
        else if (axis === 'y') number = value < 0 ? 3 : 4;
        else number = value < 0 ? 2 : 5;
    }

    
onMount(() => {
    window.addEventListener('devicemotion', (event) => {
        const g = event.accelerationIncludingGravity;
        if (!g || g.x == null || g.y == null || g.z == null) return;

        gx = g.x;
        gy = g.y;
        gz = g.z;

        const a = event.acceleration;
        if (!a || a.x == null || a.y == null || a.z == null) return;

        const strength = Math.hypot(a.x, a.y, a.z);
        const now = Date.now();

        if (strength > 15 && now - lastShake > 1000) {
            lastShake = now;
            roll();
        }
    });
});

</script>


<svelte:head>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
</svelte:head>

<main>
    <h1>Shake the Dice</h1>

    <div class="dice">
        {number ?? '–'}
    </div>

    <button onclick={start}>Start</button>
    <p>{status}</p>
</main>

<style>
    main {
        min-height: 80vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        font-family: system-ui, sans-serif;
        gap: 20px;
    }

    .dice {
        width: 180px;
        height: 180px;
        display: grid;
        place-items: center;
        border: 3px solid #222;
        border-radius: 28px;
        font-size: 100px;
        font-weight: 700;
        box-shadow: 0 8px 0 #ccc;
    }

    button {
        padding: 14px 30px;
        font-size: 18px;
        border: 0;
        border-radius: 12px;
        background: #222;
        color: white;
    }
</style>
