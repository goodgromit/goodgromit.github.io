---
title: "Parallax with Depth Map"
date: 2026-04-09T18:40:00+09:00
draft: false
---

<style>
  #parallax-container {
    width: 100%;
    max-width: 800px;
    margin: 40px auto;
    position: relative;
    overflow: hidden;
    border-radius: 8px;
    box-shadow: 0 10px 40px rgba(0,0,0,0.2);
  }
  
  #parallax-canvas {
    width: 100%;
    height: auto;
    display: block;
    cursor: move;
  }
  
  #hint {
    text-align: center;
    margin-top: 20px;
    color: #666;
    font-size: 14px;
  }
</style>

<div id="parallax-container">
  <canvas id="parallax-canvas"></canvas>
</div>
<div id="hint">Move your mouse over the image</div>

<script>
(function() {
  const canvas = document.getElementById('parallax-canvas');
  const ctx = canvas.getContext('2d');
  const hint = document.getElementById('hint');
  
  const colorImg = new Image();
  const depthImg = new Image();
  
  let loaded = 0;
  let mouseX = 0.5;
  let mouseY = 0.5;
  let targetX = 0.5;
  let targetY = 0.5;
  
  // Load images
  colorImg.onload = checkLoaded;
  depthImg.onload = checkLoaded;
  
  colorImg.src = '/images/example.webp';
  depthImg.src = '/images/example_depth.webp';
  
  function checkLoaded() {
    loaded++;
    if (loaded === 2) {
      setupCanvas();
      animate();
    }
  }
  
  function setupCanvas() {
    canvas.width = colorImg.width;
    canvas.height = colorImg.height;
    canvas.style.width = '100%';
    canvas.style.height = 'auto';
  }
  
  // Mouse/touch/gyro interaction
  const isDesktop = window.matchMedia('(min-width: 768px)').matches;
  
  if (isDesktop) {
    // Desktop: Mouse
    canvas.addEventListener('mousemove', (e) => {
      const rect = canvas.getBoundingClientRect();
      targetX = (e.clientX - rect.left) / rect.width;
      targetY = (e.clientY - rect.top) / rect.height;
    });
    
    canvas.addEventListener('mouseleave', () => {
      targetX = 0.5;
      targetY = 0.5;
    });
  } else {
    // Mobile: Gyroscope
    hint.textContent = 'Tilt your device (tap to enable)';
    
    let gyroActive = false;
    let initialBeta = null;
    let initialGamma = null;
    
    function handleOrientation(event) {
      const beta = event.beta;   // X축 회전 (-180 ~ 180)
      const gamma = event.gamma; // Y축 회전 (-90 ~ 90)
      
      if (beta === null || gamma === null) return;
      
      // 초기 각도 설정 (캘리브레이션)
      if (initialBeta === null) {
        initialBeta = beta;
        initialGamma = gamma;
        return;
      }
      
      // 초기 위치 대비 변화량
      const deltaBeta = beta - initialBeta;
      const deltaGamma = gamma - initialGamma;
      
      // -30 ~ +30도 범위를 0 ~ 1로 매핑
      targetX = 0.5 + (deltaGamma / 60);
      targetY = 0.5 + (deltaBeta / 60);
      
      // 범위 제한
      targetX = Math.max(0, Math.min(1, targetX));
      targetY = Math.max(0, Math.min(1, targetY));
    }
    
    function enableGyro() {
      if (gyroActive) return;
      
      // iOS 13+ 권한 요청
      if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        DeviceOrientationEvent.requestPermission()
          .then(state => {
            if (state === 'granted') {
              window.addEventListener('deviceorientation', handleOrientation);
              gyroActive = true;
              hint.textContent = 'Tilt your device to move';
            } else {
              hint.textContent = 'Permission denied - Touch and drag instead';
              enableTouchDrag();
            }
          })
          .catch((err) => {
            console.error(err);
            hint.textContent = 'Gyro not available - Touch and drag instead';
            enableTouchDrag();
          });
      } else {
        // Android 및 이전 iOS
        window.addEventListener('deviceorientation', handleOrientation);
        gyroActive = true;
        hint.textContent = 'Tilt your device to move';
      }
    }
    
    function enableTouchDrag() {
      hint.textContent = 'Touch and drag to move';
      let isDragging = false;
      
      canvas.addEventListener('touchstart', () => {
        isDragging = true;
      });
      
      canvas.addEventListener('touchmove', (e) => {
        if (!isDragging) return;
        e.preventDefault();
        const touch = e.touches[0];
        const rect = canvas.getBoundingClientRect();
        targetX = (touch.clientX - rect.left) / rect.width;
        targetY = (touch.clientY - rect.top) / rect.height;
      });
      
      canvas.addEventListener('touchend', () => {
        isDragging = false;
        targetX = 0.5;
        targetY = 0.5;
      });
    }
    
    // 자이로 시작 (탭 or 자동)
    if (window.DeviceOrientationEvent) {
      canvas.addEventListener('click', enableGyro, { once: true });
      
      // iOS가 아니면 자동 시작
      if (typeof DeviceOrientationEvent.requestPermission !== 'function') {
        enableGyro();
      }
    } else {
      // 자이로 없으면 터치 드래그
      enableTouchDrag();
    }
  }
  
  function animate() {
    requestAnimationFrame(animate);
    
    // Smooth lerp
    mouseX += (targetX - mouseX) * 0.1;
    mouseY += (targetY - mouseY) * 0.1;
    
    render();
  }
  
  function render() {
    const width = canvas.width;
    const height = canvas.height;
    
    // Clear canvas
    ctx.clearRect(0, 0, width, height);
    
    // Get depth data
    const tempCanvas = document.createElement('canvas');
    tempCanvas.width = width;
    tempCanvas.height = height;
    const tempCtx = tempCanvas.getContext('2d');
    tempCtx.drawImage(depthImg, 0, 0, width, height);
    const depthData = tempCtx.getImageData(0, 0, width, height);
    
    // Draw with stronger parallax effect
    const offsetX = (mouseX - 0.5) * 120; // Increased from 50 to 120
    const offsetY = (mouseY - 0.5) * 120;
    
    // Draw base image
    ctx.drawImage(colorImg, 0, 0, width, height);
    
    // Create displacement effect based on depth
    const imageData = ctx.getImageData(0, 0, width, height);
    const pixels = imageData.data;
    const depthPixels = depthData.data;
    
    const newImageData = ctx.createImageData(width, height);
    const newPixels = newImageData.data;
    
    for (let y = 0; y < height; y++) {
      for (let x = 0; x < width; x++) {
        const i = (y * width + x) * 4;
        
        // Get depth value (0-255)
        let depth = depthPixels[i] / 255;
        
        // Apply non-linear curve for stronger depth perception
        depth = Math.pow(depth, 1.5); // Stronger depth gradient
        
        // Calculate offset based on depth with enhanced effect
        const dx = Math.round(offsetX * depth);
        const dy = Math.round(offsetY * depth);
        
        // Source pixel position
        const sx = Math.max(0, Math.min(width - 1, x - dx));
        const sy = Math.max(0, Math.min(height - 1, y - dy));
        const si = (sy * width + sx) * 4;
        
        // Copy pixel
        newPixels[i] = pixels[si];
        newPixels[i + 1] = pixels[si + 1];
        newPixels[i + 2] = pixels[si + 2];
        newPixels[i + 3] = pixels[si + 3];
      }
    }
    
    ctx.putImageData(newImageData, 0, 0);
  }
})();
</script>
