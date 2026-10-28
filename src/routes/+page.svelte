
<script>
    let number = $state(null);
    let status = $state('Press Start');
    let beta = 0;
    let gamma = 0;
    let lastShake = 0;

    async function start() {
        try {
            if (typeof DeviceMotionEvent.requestPermission === 'function') {
                const permission = await DeviceMotionEvent.requestPermission();
                if (permission !== 'granted') {
                    status = 'Motion permission denied';
                    return;
                }
            }

            if (typeof DeviceOrientationEvent.requestPermission === 'function') {
                const permission = await DeviceOrientationEvent.requestPermission();
                if (permission !== 'granted') {
                    status = 'Orientation permission denied';
                    return;
                }
            }

            window.addEventListener('deviceorientation', (e) => {
                beta = e.beta ?? 0;
                gamma = e.gamma ?? 0;
            });

            window.addEventListener('devicemotion', (e) => {
                const a = e.acceleration;
                if (!a || a.x === null || a.y === null || a.z === null) return;

                const strength = Math.sqrt(
                    a.x ** 2 + a.y ** 2 + a.z ** 2
                );

                const now = Date.now();

                if (strength > 15 && now - lastShake > 1000) {
                    lastShake = now;
                    roll();
                }
            });

            status = 'Ready — shake your iPhone!';
        } catch (e) {
            status = e.message;
        }
    }

    
function roll() {
    // Screen facing up
    if (beta > -45 && beta < 45 &&
        gamma > -45 && gamma < 45) {
        number = 1;
    }
    // Screen facing down
    else if (Math.abs(beta) > 135) {
        number = 6;
    }
    // Earpiece edge up
    else if (beta >= 45 && beta < 135) {
        number = 3;
    }
    // Earpiece edge down
    else if (beta <= -45 && beta > -135) {
        number = 4;
    }
    // Side edges: distinguish using gamma
    else if (gamma >= 0) {
        number = 2;
    } else {
        number = 5;
    }
}

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
