<script lang="ts">
    import { onMount } from "svelte";

    let cats = $state(0);
    let catsPerClick = $state(1);
    let autoCats = $state(0);
    let catMultiplier = $state(1);
    let rebirthLevel = $state(0);
    let rebirthCost = $state(1000);
    let alpyLevel = $state(0);
    let alpyCost = $state(1000000);

    let baseUpgrades = [
        {
            name: "Cat Toy",
            baseCost: 5,
            cost: 5,
            effect: () => (catsPerClick += 1),
            description: "Add 1 cat per click",
        },
        {
            name: "Catnip",
            baseCost: 25,
            cost: 25,
            effect: () => (catsPerClick *= 1.5),
            description: "Increase cats per click by 50%",
        },
        {
            name: "Stray Cat",
            baseCost: 100,
            cost: 100,
            effect: () => (autoCats += 1),
            description: "Add 1 cat per second",
        },
        {
            name: "Cat Bed",
            baseCost: 250,
            cost: 250,
            effect: () => {
                catsPerClick *= 1.2;
                autoCats *= 1.2;
            },
            description: "Increase all cat production by 20%",
        },
        {
            name: "Cat Tree",
            baseCost: 500,
            cost: 500,
            effect: () => (autoCats += 5),
            description: "Add 5 cats per second",
        },
        {
            name: "Cat Cafe",
            baseCost: 1000,
            cost: 1000,
            effect: () => (autoCats += 10),
            description: "Add 10 cats per second",
        },
        {
            name: "Feline University",
            baseCost: 2500,
            cost: 2500,
            effect: () => (catMultiplier *= 1.5),
            description: "Increase all cat production by 50%",
        },
        {
            name: "Cat Cloning Facility",
            baseCost: 5000,
            cost: 5000,
            effect: () => {
                catsPerClick *= 2;
                autoCats *= 2;
            },
            description: "Double all cat production",
        },
    ];

    let rebirthUpgrades = [
        {
            name: "Time Warp Whiskers",
            baseCost: 10000,
            cost: 10000,
            effect: () => (autoCats *= 3),
            description: "Triple automatic cat production",
            unlockLevel: 1,
        },
        {
            name: "Quantum Cat Box",
            baseCost: 25000,
            cost: 25000,
            effect: () => {
                catsPerClick *= 3;
                autoCats *= 2;
            },
            description: "Triple clicks and double auto cats",
            unlockLevel: 2,
        },
        {
            name: "Cosmic Catnip",
            baseCost: 50000,
            cost: 50000,
            effect: () => (catMultiplier *= 3),
            description: "Triple all cat production",
            unlockLevel: 3,
        },
        {
            name: "Interdimensional Cat Portal",
            baseCost: 100000,
            cost: 100000,
            effect: () => {
                autoCats += 1000;
                catsPerClick *= 5;
            },
            description: "Add 1000 auto cats and 5x clicks",
            unlockLevel: 4,
        },
        {
            name: "Galactic Cat Empire",
            baseCost: 500000,
            cost: 500000,
            effect: () => {
                catMultiplier *= 5;
                autoCats *= 5;
                catsPerClick *= 5;
            },
            description: "Quintuple all cat production",
            unlockLevel: 5,
        },
    ];

    let upgrades = $state([...baseUpgrades]);

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let animationFrameId: number;

    interface CatObject {
        x: number;
        y: number;
        vx: number;
        vy: number;
        color: string;
    }

    let catObjects: CatObject[] = [];

    onMount(() => {
        ctx = canvas.getContext("2d")!;

        setInterval(() => {
            cats += autoCats * catMultiplier * (1 + alpyLevel);
        }, 1000);

        animateCanvas();

        return () => {
            cancelAnimationFrame(animationFrameId);
        };
    });

    function addCats() {
        const catsToAdd = catsPerClick * catMultiplier * (1 + alpyLevel);
        cats += catsToAdd;
        for (let i = 0; i < Math.min(catsToAdd, 10); i++) {
            addCatObject();
        }
    }

    function addCatObject() {
        const padding = 30;
        const x = padding + Math.random() * (canvas.width - 2 * padding);
        const y = padding + Math.random() * (canvas.height - 2 * padding);
        const colors = ["orange", "gray", "brown", "black", "white"];
        const color = colors[Math.floor(Math.random() * colors.length)];
        const speed = 0.5;
        const angle = Math.random() * Math.PI * 2;
        catObjects.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            color,
        });

        if (catObjects.length > 100) {
            catObjects.shift();
        }
    }

    function animateCanvas() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        catObjects.forEach((cat) => {
            cat.x += cat.vx;
            cat.y += cat.vy;

            if (cat.x < 0 || cat.x > canvas.width) cat.vx *= -1;
            if (cat.y < 0 || cat.y > canvas.height) cat.vy *= -1;

            drawCat(cat.x, cat.y, cat.color);
        });

        animationFrameId = requestAnimationFrame(animateCanvas);
    }

    function buyUpgrade(upgrade: (typeof upgrades)[number]) {
        if (cats >= upgrade.cost) {
            cats -= upgrade.cost;
            upgrade.effect();
            upgrade.cost = Math.round(upgrade.cost * 1.15);
        }
    }

    function rebirth() {
        if (cats >= rebirthCost) {
            rebirthLevel++;
            cats = 0;
            catsPerClick = 1 + rebirthLevel;
            autoCats = rebirthLevel;
            catMultiplier = 1 + rebirthLevel * 0.1;
            catObjects = [];
            rebirthCost = Math.round(rebirthCost * 3);

            upgrades = baseUpgrades.map((upgrade) => ({
                ...upgrade,
                cost: upgrade.baseCost,
            }));

            rebirthUpgrades.forEach((upgrade) => {
                if (upgrade.unlockLevel === rebirthLevel) {
                    upgrades.push({ ...upgrade });
                }
            });
        }
    }

    function alpy() {
        if (cats >= alpyCost) {
            alpyLevel++;
            cats -= alpyCost;
            alpyCost *= 1.5;
        }
    }

    function drawCat(x: number, y: number, color: string) {
        const outlineColor = "black";
        const outlineWidth = 1;

        ctx.lineWidth = outlineWidth;
        ctx.strokeStyle = outlineColor;

        ctx.beginPath();
        ctx.ellipse(x, y, 15, 10, 0, 0, 2 * Math.PI);
        ctx.fillStyle = color;
        ctx.fill();
        ctx.stroke();

        ctx.beginPath();
        ctx.arc(x - 15, y - 5, 8, 0, 2 * Math.PI);
        ctx.fill();
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x - 18, y - 12);
        ctx.lineTo(x - 21, y - 18);
        ctx.lineTo(x - 15, y - 12);
        ctx.fill();
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x - 12, y - 12);
        ctx.lineTo(x - 9, y - 18);
        ctx.lineTo(x - 15, y - 12);
        ctx.fill();
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x + 15, y);
        ctx.quadraticCurveTo(x + 25, y - 10, x + 30, y);
        ctx.lineWidth = 3;
        ctx.strokeStyle = color;
        ctx.stroke();
        ctx.lineWidth = outlineWidth;
        ctx.strokeStyle = outlineColor;
        ctx.stroke();

        ctx.beginPath();
        ctx.arc(x - 17, y - 6, 1, 0, 2 * Math.PI);
        ctx.arc(x - 13, y - 6, 1, 0, 2 * Math.PI);
        ctx.fillStyle = "black";
        ctx.fill();
    }

    $effect(() => {
        console.log(
            `You have ${cats.toFixed(2)} cats, rebirth level ${rebirthLevel}, and alpy level ${alpyLevel}`,
        );
    });
</script>

<div class="flex h-screen">
    <div class="flex-1 p-4 overflow-hidden">
        <h1 class="text-3xl font-bold mb-4">Advanced Cat Clicker Game</h1>

        <div class="mb-4">
            <p class="text-xl">Cats: {cats.toFixed(2)}</p>
            <p>
                Cats per click: {(
                    catsPerClick *
                    catMultiplier *
                    (1 + alpyLevel)
                ).toFixed(2)}
            </p>
            <p>
                Auto cats per second: {(
                    autoCats *
                    catMultiplier *
                    (1 + alpyLevel)
                ).toFixed(2)}
            </p>
            <p>Rebirth Level: {rebirthLevel}</p>
            <p>Alpy Level: {alpyLevel}</p>
        </div>

        <canvas
            bind:this={canvas}
            width="1000"
            height="600"
            class="border border-gray-300 mb-4 cursor-pointer"
            on:click={addCats}
        ></canvas>

        <button
            class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded mb-4 mr-2"
            on:click={rebirth}
            disabled={cats < rebirthCost}
        >
            Rebirth (Cost: {rebirthCost} cats)
        </button>

        <button
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded mb-4"
            on:click={alpy}
            disabled={cats < alpyCost}
        >
            Alpy (Cost: {alpyCost.toFixed(0)} cats)
        </button>
    </div>

    <div class="w-80 bg-gray-100 p-4 overflow-y-auto">
        <h2 class="text-2xl font-bold mb-4">Upgrades</h2>
        <div class="space-y-4">
            {#each upgrades as upgrade}
                <button
                    class="w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition duration-300 ease-in-out transform hover:scale-105"
                    on:click={() => buyUpgrade(upgrade)}
                    disabled={cats < upgrade.cost}
                >
                    <div class="text-lg">{upgrade.name}</div>
                    <div class="text-sm">
                        Cost: {upgrade.cost.toFixed(0)} cats
                    </div>
                    <div class="text-xs mt-1">{upgrade.description}</div>
                </button>
            {/each}
        </div>
    </div>
</div>
