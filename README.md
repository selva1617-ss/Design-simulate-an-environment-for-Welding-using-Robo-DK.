# Design-simulate-an-environment-for-Welding-using-Robo-DK.
Design &amp; simulate an environment for  Welding using Robo DK.
# Aim
To create a robot welding environment using RoboDK software
# Apparatus / Software required
1. Laptop / Desktop 2. Robodk software
## Procedure

### A. Straight-Line Interpolation

1. Open **RoboDK** and load the Doosan Industrial Cobot model from the RoboDK library.
2. Load the required work station containing the robot, table, workpiece, and end-effector.
3. Select the Doosan robot from the station tree and verify that the robot is positioned correctly.
4. Set the **Reference Frame** to the workpiece frame so that the robot positions are defined relative to the working area.
5. Select the required **Tool (TCP)** attached to the robot. In the loaded station, the RobotiQ sanding tool is used.
6. Move the robot manually using the RoboDK jog controls and position the TCP at the required starting location.
7. Create a target named **P1** by teaching the current robot position.
8. Move the robot to the next required position and create another target named **P2**.
9. Move the robot to a third position and create target **P3** if multiple straight-line segments are required.
10. Create a new robot program and name it **Straight-line**.
11. Add a **MoveJ** instruction to move the robot from its initial position to `P1`.
12. Add a **MoveL** instruction from `P1` to `P2`. The `MoveL` command ensures that the TCP travels along a straight-line trajectory.
13. Add another `MoveL` instruction from `P2` to `P3`, if required.
14. Run the program using the RoboDK simulation controls.
15. Observe the TCP trajectory and verify that it remains straight between the specified targets.

### B. Circular Interpolation

1. Create three different targets for defining the circular path.
2. Position the robot at the starting location and create the target **P1_Circle**.
3. Move the TCP to a suitable intermediate location on the required circular path and create **P2_Circle**.
4. Move the TCP to the final location and create **P3_Circle**.
5. Ensure that the three points are **not in a straight line**, otherwise a proper circular arc cannot be generated.
6. Keep the tool orientation consistent where appropriate so that the end-effector maintains the desired orientation during the motion.
7. Create a new program named **Circular_Interpolation**.
8. Add a **MoveJ** instruction to move the robot to `P1_Circle`.
9. Add a **MoveC** instruction and select `P2_Circle` as the intermediate point and `P3_Circle` as the final point.
10. RoboDK calculates the circular trajectory passing through the three specified positions.
11. Run the program and observe the movement of the Doosan robot.
12. Verify that the TCP moves from `P1_Circle`, passes through `P2_Circle`, and ends at `P3_Circle` following a curved path.
13. Adjust the positions of the three targets if a larger, smaller, or differently oriented circular arc is required.
14. Run the simulation again and verify the final trajectory.

---
---
# Program 

```
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Doosan Cobot - Interpolation Simulation</title>

<style>
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: #111827;
        color: white;
    }

    header {
        padding: 20px;
        text-align: center;
        background: #1f2937;
    }

    h1 {
        margin: 0;
        font-size: 26px;
    }

    .container {
        display: flex;
        gap: 20px;
        padding: 20px;
    }

    .panel {
        width: 280px;
        background: #1f2937;
        padding: 20px;
        border-radius: 12px;
    }

    .simulation {
        flex: 1;
        background: #0f172a;
        border-radius: 12px;
        position: relative;
        min-height: 550px;
        overflow: hidden;
    }

    button {
        width: 100%;
        padding: 12px;
        margin: 7px 0;
        border: none;
        border-radius: 7px;
        cursor: pointer;
        font-size: 15px;
    }

    .linear {
        background: #2563eb;
        color: white;
    }

    .circular {
        background: #16a34a;
        color: white;
    }

    .stop {
        background: #dc2626;
        color: white;
    }

    button:hover {
        opacity: 0.85;
    }

    canvas {
        width: 100%;
        height: 100%;
    }

    .info {
        margin-top: 20px;
        padding: 12px;
        background: #374151;
        border-radius: 8px;
        font-size: 14px;
        line-height: 1.6;
    }

    .status {
        margin-top: 10px;
        padding: 10px;
        border-radius: 6px;
        background: #111827;
    }
</style>
</head>

<body>

<header>
    <h1>Doosan Industrial Cobot</h1>
    <p>Straight-Line & Circular Interpolation Simulation</p>
</header>

<div class="container">

    <div class="panel">

        <h2>Robot Program</h2>

        <button class="linear" onclick="startLinear()">
            ▶ Straight-Line Interpolation
        </button>

        <button class="circular" onclick="startCircular()">
             Circular Interpolation
        </button>

        <button class="stop" onclick="stopRobot()">
            ■ Stop
        </button>

        <div class="info">
            <b>Reference Frame:</b> Part<br>
            <b>Tool:</b> RobotiQ Sanding Tool<br>
            <b>Robot:</b> Doosan Cobot
        </div>

        <div class="status">
            <b>Status:</b>
            <div id="status">Ready</div>
        </div>

        <div class="info">
            <b>Targets</b><br><br>

            P1 = Starting Point<br>
            P2 = Intermediate Point<br>
            P3 = Ending Point

            <br><br>

            <b>Linear:</b><br>
            MoveJ → P1<br>
            MoveL → P2<br>
            MoveL → P3

            <br><br>

            <b>Circular:</b><br>
            MoveJ → P1<br>
            MoveC → P2 → P3
        </div>

    </div>

    <div class="simulation">
        <canvas id="canvas"></canvas>
    </div>

</div>

<script>

const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

canvas.width = 900;
canvas.height = 550;

let animation;
let running = false;

let robot = {
    x: 150,
    y: 400
};

const P1 = {
    x: 150,
    y: 400
};

const P2 = {
    x: 450,
    y: 180
};

const P3 = {
    x: 750,
    y: 400
};


// Draw complete simulation
function drawScene(pathType) {

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Work table
    ctx.fillStyle = "#374151";
    ctx.fillRect(80, 430, 740, 50);

    // Workpiece
    ctx.fillStyle = "#9ca3af";
    ctx.fillRect(250, 350, 400, 80);

    // Draw path
    ctx.lineWidth = 5;

    if (pathType === "linear") {

        ctx.strokeStyle = "#2563eb";

        ctx.beginPath();
        ctx.moveTo(P1.x, P1.y);
        ctx.lineTo(P2.x, P2.y);
        ctx.lineTo(P3.x, P3.y);
        ctx.stroke();

    }

    if (pathType === "circular") {

        ctx.strokeStyle = "#16a34a";

        ctx.beginPath();

        ctx.moveTo(P1.x, P1.y);

        // Quadratic curve through P2
        ctx.quadraticCurveTo(
            P2.x,
            P2.y,
            P3.x,
            P3.y
        );

        ctx.stroke();
    }

    // Targets
    drawPoint(P1, "P1");
    drawPoint(P2, "P2");
    drawPoint(P3, "P3");

    // Robot
    drawRobot(robot.x, robot.y);
}


// Draw target
function drawPoint(point, label) {

    ctx.beginPath();

    ctx.arc(
        point.x,
        point.y,
        8,
        0,
        Math.PI * 2
    );

    ctx.fillStyle = "white";
    ctx.fill();

    ctx.fillStyle = "white";
    ctx.font = "16px Arial";

    ctx.fillText(
        label,
        point.x + 12,
        point.y - 10
    );
}


// Draw simple robot
function drawRobot(x, y) {

    // Base
    ctx.fillStyle = "#6b7280";

    ctx.fillRect(
        x - 30,
        y + 15,
        60,
        15
    );

    // Robot arm
    ctx.strokeStyle = "#e5e7eb";
    ctx.lineWidth = 18;

    ctx.beginPath();

    ctx.moveTo(x, y + 15);
    ctx.lineTo(x - 40, y - 60);
    ctx.lineTo(x + 20, y - 110);
    ctx.stroke();

    // Joints
    ctx.fillStyle = "#111827";

    ctx.beginPath();
    ctx.arc(x, y + 15, 10, 0, Math.PI * 2);
    ctx.fill();

    ctx.beginPath();
    ctx.arc(x - 40, y - 60, 10, 0, Math.PI * 2);
    ctx.fill();

    ctx.beginPath();
    ctx.arc(x + 20, y - 110, 10, 0, Math.PI * 2);
    ctx.fill();

    // TCP
    ctx.fillStyle = "#f59e0b";

    ctx.beginPath();
    ctx.arc(x, y, 12, 0, Math.PI * 2);
    ctx.fill();
}


// Linear interpolation
function startLinear() {

    stopRobot();

    running = true;

    document.getElementById("status").innerText =
        "Executing Straight-Line Interpolation";

    let startTime = performance.now();

    function animate(time) {

        if (!running) return;

        let elapsed = time - startTime;

        let duration = 4000;

        let t = Math.min(elapsed / duration, 1);

        if (t < 0.5) {

            let localT = t * 2;

            robot.x =
                P1.x + (P2.x - P1.x) * localT;

            robot.y =
                P1.y + (P2.y - P1.y) * localT;

        } else {

            let localT = (t - 0.5) * 2;

            robot.x =
                P2.x + (P3.x - P2.x) * localT;

            robot.y =
                P2.y + (P3.y - P2.y) * localT;
        }

        drawScene("linear");

        if (t < 1) {

            animation =
                requestAnimationFrame(animate);

        } else {

            document.getElementById("status").innerText =
                "Straight-Line Interpolation Completed";

            running = false;
        }
    }

    requestAnimationFrame(animate);
}


// Circular interpolation
function startCircular() {

    stopRobot();

    running = true;

    document.getElementById("status").innerText =
        "Executing Circular Interpolation";

    let startTime = performance.now();

    function animate(time) {

        if (!running) return;

        let elapsed = time - startTime;

        let duration = 4000;

        let t = Math.min(elapsed / duration, 1);

        /*
            Quadratic Bezier equation

            P(t) =
            (1-t)^2 P1
            + 2(1-t)t P2
            + t^2 P3
        */

        let x =
            Math.pow(1 - t, 2) * P1.x +
            2 * (1 - t) * t * P2.x +
            Math.pow(t, 2) * P3.x;

        let y =
            Math.pow(1 - t, 2) * P1.y +
            2 * (1 - t) * t * P2.y +
            Math.pow(t, 2) * P3.y;

        robot.x = x;
        robot.y = y;

        drawScene("circular");

        if (t < 1) {

            animation =
                requestAnimationFrame(animate);

        } else {

            document.getElementById("status").innerText =
                "Circular Interpolation Completed";

            running = false;
        }
    }

    requestAnimationFrame(animate);
}


// Stop
function stopRobot() {

    running = false;

    if (animation) {

        cancelAnimationFrame(animation);
    }

    document.getElementById("status").innerText =
        "Robot Stopped";
}


// Initial display
drawScene("linear");

</script>

</body>
</html>
```
# Output
<img width="1917" height="995" alt="image" src="https://github.com/user-attachments/assets/6d38ea57-278f-4956-bd51-0e9d32110e6f" />

<img width="1371" height="802" alt="image" src="https://github.com/user-attachments/assets/329c4ab1-586e-4568-a8fd-7aa7f15902f0" />
<img width="1896" height="957" alt="image" src="https://github.com/user-attachments/assets/e021ac6d-629f-432b-98c6-e3032bd59693" />

# Result

The **straight-line and circular interpolation programs were successfully created and simulated in RoboDK using the Doosan Industrial Cobot**. In the straight-line program, the robot TCP travelled along a linear path between the specified targets using `MoveL`. In the circular interpolation program, the robot travelled from `P1_Circle` through `P2_Circle` to `P3_Circle` along a curved circular trajectory using `MoveC`. The robot movements and corresponding trajectories were successfully observed in the RoboDK simulation.
