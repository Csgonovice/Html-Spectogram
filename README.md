<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Audio Spectrogram</title>
    <style>
        body { margin: 0; background: black; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        button { font-size: 20px; padding: 10px 20px; cursor: pointer; position: relative; z-index: 1; }
        canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }
        #hzDisplay { position: absolute; top: 10px; right: 10px; color: white; font-size: 16px; font-family: Arial, sans-serif; }
        #loopToggle { position: absolute; top: 10px; left: 10px; color: white; font-size: 10px; font-family: Arial, sans-serif; cursor: pointer; }
        #spectrogramToggle { position: absolute; top: 10px; left: 50%; transform: translateX(-50%); color: white; font-size: 10px; font-family: Arial, sans-serif; cursor: pointer; }
    </style>
</head>
<body>
    <button id="selectFile">Select MP3</button>
    <input type="file" id="fileInput" accept="audio/mp3" style="display: none;">
    <canvas id="spectrogram"></canvas>
    <div id="hzDisplay">Hz: 0</div>
    <div id="loopToggle">Loop: OFF</div>
    <div id="spectrogramToggle">Spectrogram: ON</div>
    
    <script>
        const button = document.getElementById('selectFile');
        const fileInput = document.getElementById('fileInput');
        const canvas = document.getElementById('spectrogram');
        const ctx = canvas.getContext('2d');
        const hzDisplay = document.getElementById('hzDisplay');
        const loopToggle = document.getElementById('loopToggle');
        const spectrogramToggle = document.getElementById('spectrogramToggle');
        let audioContext, analyser, source, gainNode, audio;
        let isLooping = false;
        let isSpectrogramVisible = true;
        
        button.addEventListener('click', () => fileInput.click());
        fileInput.addEventListener('change', event => {
            if (event.target.files.length > 0) {
                button.remove();
                playAudio(event.target.files[0]);
            }
        });
        
        loopToggle.addEventListener('click', () => {
            isLooping = !isLooping;
            loopToggle.textContent = `Loop: ${isLooping ? 'ON' : 'OFF'}`;
            if (audio) {
                audio.loop = isLooping;
            }
        });
        
        spectrogramToggle.addEventListener('click', () => {
            isSpectrogramVisible = !isSpectrogramVisible;
            spectrogramToggle.textContent = `Spectrogram: ${isSpectrogramVisible ? 'ON' : 'OFF'}`;
            canvas.style.display = isSpectrogramVisible ? 'block' : 'none';
        });
        
        function playAudio(file) {
            const objectURL = URL.createObjectURL(file);
            audio = new Audio(objectURL);
            audio.autoplay = true;
            audio.loop = isLooping;
            
            audioContext = new (window.AudioContext || window.webkitAudioContext)();
            analyser = audioContext.createAnalyser();
            analyser.fftSize = 2048;
            
            gainNode = audioContext.createGain();
            gainNode.gain.value = 1.5; // Increase volume slightly
            
            source = audioContext.createMediaElementSource(audio);
            source.connect(gainNode);
            gainNode.connect(analyser);
            analyser.connect(audioContext.destination);
            
            resizeCanvas();
            drawSpectrogram();
        }
        
        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        
        function drawSpectrogram() {
            const bufferLength = analyser.frequencyBinCount;
            const dataArray = new Uint8Array(bufferLength);
            
            function render() {
                if (!isSpectrogramVisible) {
                    requestAnimationFrame(render);
                    return;
                }
                analyser.getByteFrequencyData(dataArray);
                ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                const barWidth = (canvas.width / bufferLength) * 2.5;
                let x = 0;
                let maxFrequency = 0;
                
                for (let i = 0; i < bufferLength; i++) {
                    const barHeight = dataArray[i] * 2;
                    if (dataArray[i] > maxFrequency) {
                        maxFrequency = dataArray[i];
                    }
                    ctx.fillStyle = `hsl(${(i / bufferLength) * 360}, 100%, 50%)`;
                    ctx.fillRect(x, canvas.height - barHeight, barWidth, barHeight);
                    x += barWidth + 1;
                }
                
                hzDisplay.textContent = `Hz: ${maxFrequency}`;
                requestAnimationFrame(render);
            }
            render();
        }
        
        window.addEventListener('resize', resizeCanvas);
    </script>
</body>
</html>
