<!--Distinction between human body keypoints that can't be moved and bike keypoints that be dragged and moved. Human keypoints are the ankle, the knee, the hip, the elbow and the wrist (bottom up). Bike keypoints are the saddle and the handlebar.-->
<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D;

	// Cyclist pose points (relative to canvas size)
	let posePoints: { [key: string]: { x: number; y: number } } = {
		shoulder: { x: 0.35, y: 0.25 },
		elbow: { x: 0.45, y: 0.3 },
		hand: { x: 0.55, y: 0.35 },
		hip: { x: 0.4, y: 0.4 },
		knee: { x: 0.3, y: 0.65 },
		ankle: { x: 0.35, y: 0.8 },
		saddle: { x: 0.4, y: 0.42 },
		handlebar: { x: 0.55, y: 0.35 }
	};

	let dragging: string | null = null;
	let hovering: string | null = null;

	// Define keypoint categories based on your comment
	const humanBodyKeypoints = ['ankle', 'knee', 'hip', 'elbow', 'hand']; // Human body points (read-only)
	const bikeKeypoints = ['saddle', 'handlebar']; // Bike components (adjustable)
	const allKeypoints = [...humanBodyKeypoints, ...bikeKeypoints];

	onMount(() => {
		ctx = canvas.getContext('2d')!;
		drawCyclist();

		// Add event listeners for interaction
		canvas.addEventListener('mousedown', handleMouseDown);
		canvas.addEventListener('mousemove', handleMouseMove);
		canvas.addEventListener('mouseup', handleMouseUp);
		canvas.addEventListener('mouseleave', handleMouseUp);
	});

    function drawCyclist() {
        if (!ctx) return;

        // Simple dark background
        ctx.fillStyle = '#0f1419';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        const width = canvas.width;
        const height = canvas.height;

        // Convert relative positions to absolute
        const points: { [key: string]: { x: number; y: number } } = {};
        Object.entries(posePoints).forEach(([key, point]) => {
            points[key] = { x: point.x * width, y: point.y * height };
        });

        // Calculate bike frame points based on current saddle and handlebar positions
        const wheelBase = Math.abs(points.saddle.x - points.handlebar.x) + 40; // Distance between wheels
        const frontWheel = {
            x: points.handlebar.x + 30,
            y: Math.max(points.ankle.y, points.saddle.y + 80)
        };
        const rearWheel = { x: points.saddle.x - 30, y: frontWheel.y };
        const bottomBracket = { x: (frontWheel.x + rearWheel.x) / 2 - 10, y: frontWheel.y - 25 };

        // Calculate pedal position and update ankle to match
        const pedalX = bottomBracket.x - 12;
        const pedalY = bottomBracket.y + 8;
        
        // Keep ankle connected to pedal
        posePoints.ankle.x = pedalX / width;
        posePoints.ankle.y = pedalY / height;
        points.ankle = { x: pedalX, y: pedalY };

        // Draw bike frame - more realistic geometry
        ctx.strokeStyle = '#4a5568';
        ctx.lineWidth = 2;

        // Draw main triangle frame
        ctx.beginPath();
        ctx.moveTo(points.saddle.x, points.saddle.y); // Saddle
        ctx.lineTo(points.handlebar.x, points.handlebar.y); // Handlebar stem
        ctx.lineTo(bottomBracket.x, bottomBracket.y); // Bottom bracket
        ctx.lineTo(points.saddle.x, points.saddle.y); // Back to saddle
        ctx.stroke();

        // Draw seat tube
        ctx.beginPath();
        ctx.moveTo(points.saddle.x, points.saddle.y);
        ctx.lineTo(bottomBracket.x, bottomBracket.y);
        ctx.stroke();

        // Draw chain stays
        ctx.beginPath();
        ctx.moveTo(bottomBracket.x, bottomBracket.y);
        ctx.lineTo(rearWheel.x, rearWheel.y);
        ctx.stroke();

        // Draw fork
        ctx.beginPath();
        ctx.moveTo(points.handlebar.x, points.handlebar.y);
        ctx.lineTo(frontWheel.x, frontWheel.y);
        ctx.stroke();

        // Draw wheels
        ctx.beginPath();
        ctx.arc(frontWheel.x, frontWheel.y, 35, 0, 2 * Math.PI);
        ctx.stroke();
        ctx.beginPath();
        ctx.arc(rearWheel.x, rearWheel.y, 35, 0, 2 * Math.PI);
        ctx.stroke();

        // Draw pedals - highlight the one connected to ankle
        ctx.strokeStyle = '#4a5568';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(bottomBracket.x - 12, bottomBracket.y + 8, 6, 0, 2 * Math.PI);
        ctx.stroke();
        
        // Highlight the pedal that's connected to the ankle
        ctx.fillStyle = '#f59e0b';
        ctx.beginPath();
        ctx.arc(bottomBracket.x - 12, bottomBracket.y + 8, 4, 0, 2 * Math.PI);
        ctx.fill();
        
        ctx.strokeStyle = '#4a5568';
        ctx.beginPath();
        ctx.arc(bottomBracket.x + 12, bottomBracket.y - 8, 6, 0, 2 * Math.PI);
        ctx.stroke();

        // Draw cyclist body - realistic cycling position
        ctx.strokeStyle = '#e2e8f0';
        ctx.lineWidth = 3;
        ctx.lineCap = 'round';

        // Body segments - more natural cycling pose
        const segments = [
            [points.ankle, points.knee],
            [points.knee, points.hip], 
            [points.hip, points.shoulder], 
            [points.shoulder, points.elbow], 
            [points.elbow, points.hand], 
        ];

        segments.forEach(([start, end]) => {
            ctx.beginPath();
            ctx.moveTo(start.x, start.y);
            ctx.lineTo(end.x, end.y);
            ctx.stroke();
        });

        // Draw key points with clear distinction between human body and bike keypoints
        Object.entries(points).forEach(([key, point]) => {
            const isHumanBodyPoint = humanBodyKeypoints.includes(key);
            const isBikePoint = bikeKeypoints.includes(key);

            if (!isHumanBodyPoint && !isBikePoint) return;

            // Different styling based on keypoint type
            let fillColor = '#94a3b8';
            let strokeColor = 'rgba(255, 255, 255, 0.3)';
            let size = 6;

            if (isBikePoint) {
                // Bike keypoints (adjustable) - Orange/Yellow theme
                fillColor = '#f59e0b'; // Orange
                strokeColor = 'rgba(245, 158, 11, 0.8)';
                size = 9;

                if (hovering === key) {
                    fillColor = '#fbbf24'; // Lighter orange on hover
                    size = 12;
                    // Draw hover area indicator for draggable points
                    ctx.strokeStyle = 'rgba(251, 191, 36, 0.4)';
                    ctx.lineWidth = 3;
                    ctx.beginPath();
                    ctx.arc(point.x, point.y, 28, 0, 2 * Math.PI);
                    ctx.stroke();
                } else if (dragging === key) {
                    fillColor = '#d97706'; // Darker orange when dragging
                    size = 12;
                }
            } else if (isHumanBodyPoint) {
                // Human body keypoints (read-only) - Blue/Gray theme
                fillColor = '#64748b'; // Gray-blue
                strokeColor = 'rgba(100, 116, 139, 0.6)';
                size = 6;

                if (hovering === key) {
                    fillColor = key === 'ankle' ? '#a78bfa' : '#94a3b8'; // Lighter color on hover
                    size = key === 'ankle' ? 9 : 8;
                    // Show that these points are not draggable (except ankle is auto-positioned)
                    ctx.strokeStyle = key === 'ankle' ? 'rgba(167, 139, 250, 0.3)' : 'rgba(148, 163, 184, 0.3)';
                    ctx.lineWidth = 2;
                    ctx.setLineDash([4, 4]);
                    ctx.beginPath();
                    ctx.arc(point.x, point.y, 20, 0, 2 * Math.PI);
                    ctx.stroke();
                    ctx.setLineDash([]);
                }
            }

            // Draw point
            ctx.fillStyle = fillColor;
            ctx.beginPath();
            ctx.arc(point.x, point.y, size, 0, 2 * Math.PI);
            ctx.fill();

            // Draw outline
            ctx.strokeStyle = strokeColor;
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.arc(point.x, point.y, size, 0, 2 * Math.PI);
            ctx.stroke();

        });

        drawAngle(points.hip, points.knee, points.ankle, 'knee');
    }

	function drawAngle(
		p1: { x: number; y: number },
		vertex: { x: number; y: number },
		p3: { x: number; y: number },
		label: string
	) {
		const angle = calculateAngle(p1, vertex, p3);

		ctx.strokeStyle = '#64748b';
		ctx.lineWidth = 1;
		ctx.setLineDash([3, 3]);

		// Draw subtle angle arc
		const radius = 25;
		const angle1 = Math.atan2(p1.y - vertex.y, p1.x - vertex.x);
		const angle2 = Math.atan2(p3.y - vertex.y, p3.x - vertex.x);

		ctx.beginPath();
		ctx.arc(vertex.x, vertex.y, radius, angle1, angle2);
		ctx.stroke();
		ctx.setLineDash([]);

		// Simple angle text
		ctx.fillStyle = '#94a3b8';
		ctx.font = '11px system-ui';
		ctx.fillText(`${Math.round(angle)}°`, vertex.x + 30, vertex.y);
	}

	function calculateAngle(
		p1: { x: number; y: number },
		vertex: { x: number; y: number },
		p3: { x: number; y: number }
	): number {
		const angle1 = Math.atan2(p1.y - vertex.y, p1.x - vertex.x);
		const angle2 = Math.atan2(p3.y - vertex.y, p3.x - vertex.x);
		let angle = Math.abs(angle1 - angle2) * (180 / Math.PI);
		return angle > 180 ? 360 - angle : angle;
	}

	function getPointAtMouse(e: MouseEvent): string | null {
		const rect = canvas.getBoundingClientRect();
		const scaleX = canvas.width / rect.width;
		const scaleY = canvas.height / rect.height;

		const mouseX = (e.clientX - rect.left) * scaleX;
		const mouseY = (e.clientY - rect.top) * scaleY;

		// Only check bike keypoints for interaction (as per your comment)
		for (const pointName of bikeKeypoints) {
			const point = posePoints[pointName];
			const px = point.x * canvas.width;
			const py = point.y * canvas.height;

			const distance = Math.sqrt((mouseX - px) ** 2 + (mouseY - py) ** 2);
			if (distance < 30) {
				// Larger hit area for bike points
				return pointName;
			}
		}

		// Also check human body points for hover effects (but not dragging)
		for (const pointName of humanBodyKeypoints) {
			const point = posePoints[pointName];
			const px = point.x * canvas.width;
			const py = point.y * canvas.height;

			const distance = Math.sqrt((mouseX - px) ** 2 + (mouseY - py) ** 2);
			if (distance < 20) {
				// Smaller hit area for body points
				return pointName;
			}
		}

		return null;
	}

	function handleMouseDown(e: MouseEvent) {
		const point = getPointAtMouse(e);
		if (point && bikeKeypoints.includes(point)) {
			// Only allow dragging of bike keypoints
			dragging = point;
			canvas.style.cursor = 'grabbing';
			e.preventDefault();
		}
	}

	function handleMouseMove(e: MouseEvent) {
		if (dragging) {
			const rect = canvas.getBoundingClientRect();
			const scaleX = canvas.width / rect.width;
			const scaleY = canvas.height / rect.height;

			const x = ((e.clientX - rect.left) * scaleX) / canvas.width;
			const y = ((e.clientY - rect.top) * scaleY) / canvas.height;

			// Constrain to canvas bounds
			const newX = Math.max(0.1, Math.min(0.9, x));
			const newY = Math.max(0.1, Math.min(0.9, y));

			posePoints[dragging].x = newX;
			posePoints[dragging].y = newY;

			// When moving saddle or handlebar, keep cyclist connected to bike
			if (dragging === 'saddle') {
				// Keep hip close to saddle
				posePoints.hip.x = newX;
				posePoints.hip.y = newY - 0.02;
			} else if (dragging === 'handlebar') {
				// Keep hands on handlebar
				posePoints.hand.x = newX;
				posePoints.hand.y = newY;
			}

			drawCyclist();
			return;
		}

		const point = getPointAtMouse(e);
		hovering = point;

		// Set different cursors based on point type
		if (point) {
			if (bikeKeypoints.includes(point)) {
				canvas.style.cursor = 'grab'; // Draggable bike points
			} else if (humanBodyKeypoints.includes(point)) {
				canvas.style.cursor = 'not-allowed'; // Non-draggable body points
			}
		} else {
			canvas.style.cursor = 'default';
		}

		drawCyclist();
	}

	function handleMouseUp() {
		dragging = null;
		canvas.style.cursor = hovering ? 'grab' : 'default';
	}

	function resetPose() {
		posePoints = {
			shoulder: { x: 0.35, y: 0.25 },
			elbow: { x: 0.45, y: 0.3 },
			hand: { x: 0.55, y: 0.35 },
			hip: { x: 0.4, y: 0.4 },
			knee: { x: 0.3, y: 0.65 },
			ankle: { x: 0.35, y: 0.8 },
			saddle: { x: 0.4, y: 0.42 },
			handlebar: { x: 0.55, y: 0.35 }
		};
		drawCyclist();
	}
</script>

<div class="min-h-screen bg-gray-900 p-8">
	<div class="mx-auto max-w-5xl">
		<!-- Minimal Header -->
		<div class="mb-16 text-center">
			<h1 class="mb-3 text-3xl font-light text-gray-100">Bike Fitting</h1>
			<p class="text-sm text-gray-400">Interactive pose analysis</p>
		</div>

		<!-- Main Content -->
		<div class="mb-8 rounded-lg border border-gray-700 bg-gray-800 p-6">
			<div class="grid gap-8 lg:grid-cols-4">
				<!-- Cyclist Visualization -->
				<div class="lg:col-span-3">
					<div class="mb-4 flex items-center justify-between">
						<h2 class="text-lg font-medium text-gray-200">Analysis</h2>
						<button
							on:click={resetPose}
							class="rounded bg-gray-700 px-4 py-2 text-sm text-gray-300 transition-colors hover:bg-gray-600"
						>
							Reset
						</button>
					</div>
					<canvas
						bind:this={canvas}
						width="600"
						height="400"
						class="w-full rounded border border-gray-700"
					></canvas>
					<p class="mt-3 text-xs text-gray-500">
						🔧 <span class="text-orange-400">Orange points</span> = Adjustable bike components •
						<span class="text-gray-400">Gray points</span> = Human body keypoints (read-only)
					</p>
				</div>

				<!-- Metrics Panel -->
				<div class="space-y-4">
					<div>
						<h3 class="mb-3 text-sm font-medium text-gray-300">Metrics</h3>
						<div class="space-y-3">
							<div class="rounded bg-gray-900 p-3">
								<div class="mb-1 flex items-center justify-between">
									<span class="text-xs text-gray-400">Knee angle</span>
									<span class="text-sm text-gray-200">142°</span>
								</div>
								<div class="h-1 w-full rounded-full bg-gray-700">
									<div class="h-1 rounded-full bg-blue-500" style="width: 75%"></div>
								</div>
							</div>

							<div class="rounded bg-gray-900 p-3">
								<div class="mb-1 flex items-center justify-between">
									<span class="text-xs text-gray-400">Hip angle</span>
									<span class="text-sm text-gray-200">89°</span>
								</div>
								<div class="h-1 w-full rounded-full bg-gray-700">
									<div class="h-1 rounded-full bg-yellow-500" style="width: 60%"></div>
								</div>
							</div>

							<div class="rounded bg-gray-900 p-3">
								<div class="mb-1 flex items-center justify-between">
									<span class="text-xs text-gray-400">Efficiency</span>
									<span class="text-sm text-gray-200">94%</span>
								</div>
								<div class="h-1 w-full rounded-full bg-gray-700">
									<div class="h-1 rounded-full bg-green-500" style="width: 85%"></div>
								</div>
							</div>
						</div>
					</div>

					<div>
						<h3 class="mb-3 text-sm font-medium text-gray-300">Notes</h3>
						<div class="space-y-2 text-xs text-gray-400">
							<p>• Good saddle position</p>
							<p>• Consider handlebar height</p>
							<p>• Neutral knee tracking</p>
						</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Simple Actions -->
		<div class="flex justify-center space-x-4">
			<button
				class="rounded bg-blue-600 px-6 py-2 text-sm text-white transition-colors hover:bg-blue-700"
			>
				Upload Video
			</button>
			<button
				class="rounded bg-gray-700 px-6 py-2 text-sm text-gray-300 transition-colors hover:bg-gray-600"
			>
				Save Analysis
			</button>
		</div>
	</div>
</div>
