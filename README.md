<!DOCTYPE html>
<html lang="he">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>הקלטת קול וניתוח תדרים</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 20px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            margin: 10px;
            cursor: pointer;
        }
        canvas {
            border: 1px solid black;
        }
    </style>
</head>
<body>
    <h1>הקלטת קול וניתוח תדרים</h1>
    <button id="recordButton">הקלט</button>
    <button id="stopButton" disabled>עצור</button>
    <canvas id="frequencyCanvas" width="800" height="200"></canvas>

    <script>
        const recordButton = document.getElementById('recordButton');
        const stopButton = document.getElementById('stopButton');
        const frequencyCanvas = document.getElementById('frequencyCanvas');
        const canvasCtx = frequencyCanvas.getContext('2d');

        let audioContext;
        let analyser;
        let mediaRecorder;
        let audioChunks = [];

        recordButton.addEventListener('click', async () => {
            audioContext = new (window.AudioContext || window.webkitAudioContext)();
            analyser = audioContext.createAnalyser();
            analyser.fftSize = 2048;
            const dataArray = new Uint8Array(analyser.frequencyBinCount);

            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                const source = audioContext.createMediaStreamSource(stream);
                source.connect(analyser);

                mediaRecorder = new MediaRecorder(stream);
                mediaRecorder.ondataavailable = event => {
                    audioChunks.push(event.data);
                };

                mediaRecorder.onstop = () => {
                    audioChunks = [];
                };

                mediaRecorder.start();
                recordButton.disabled = true;
                stopButton.disabled = false;

                function drawFrequency() {
                    analyser.getByteFrequencyData(dataArray);
                    canvasCtx.fillStyle = 'rgb(0, 0, 0)';
                    canvasCtx.fillRect(0, 0, frequencyCanvas.width, frequencyCanvas.height);

                    const barWidth = (frequencyCanvas.width / analyser.frequencyBinCount) * 2.5;
                    let x = 0;

                    for (let i = 0; i < analyser.frequencyBinCount; i++) {
                        const barHeight = dataArray[i];

                        canvasCtx.fillStyle = rgb(${barHeight + 100}, 50, 50);
                        canvasCtx.fillRect(x, frequencyCanvas.height - barHeight / 2, barWidth, barHeight / 2);

                        x += barWidth + 1;
                    }

                    requestAnimationFrame(drawFrequency);
                }

                drawFrequency();
            } catch (error) {
                console.error('שגיאה בהקלטת שמע:', error);
                alert('לא ניתן לגשת למיקרופון.');
            }
        });

        stopButton.addEventListener('click', () => {
            mediaRecorder.stop();
            recordButton.disabled = false;
            stopButton.disabled = true;
        });
    </script>
</body>
</html>
