---
title: "Picture"
date: 2026-04-09T13:42:00+09:00
draft: false
---

<div style="perspective: 1000px; display: flex; justify-content: center; margin: 40px 0;">
  <img id="perspective-img" src="/images/example.webp" alt="Example Image" style="transform: rotateY(0deg) rotateX(0deg); box-shadow: 20px 20px 40px rgba(0,0,0,0.3); max-width: 600px; width: 100%; border-radius: 8px; transition: transform 0.1s ease-out;">
</div>

<script>
(function() {
  const img = document.getElementById('perspective-img');
  let isDesktop = window.matchMedia('(min-width: 768px)').matches;
  
  // 모바일: 초기 각도 저장
  let initialGamma = null;
  let initialBeta = null;
  let calibrated = false;
  
  // 데스크톱: 마우스 이벤트
  if (isDesktop) {
    document.addEventListener('mousemove', function(e) {
      const x = e.clientX / window.innerWidth;
      const y = e.clientY / window.innerHeight;
      
      const rotateY = (x - 0.5) * 60; // -30도 ~ 30도
      const rotateX = (y - 0.5) * -40; // -20도 ~ 20도
      
      img.style.transform = `rotateY(${rotateY}deg) rotateX(${rotateX}deg)`;
    });
  } 
  // 모바일: 자이로센서
  else {
    if (window.DeviceOrientationEvent) {
      // iOS 13+ 권한 요청
      if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        img.addEventListener('click', function() {
          DeviceOrientationEvent.requestPermission()
            .then(permissionState => {
              if (permissionState === 'granted') {
                window.addEventListener('deviceorientation', handleOrientation);
              }
            })
            .catch(console.error);
        }, { once: true });
      } else {
        // Android 및 이전 iOS
        window.addEventListener('deviceorientation', handleOrientation);
      }
    }
  }
  
  function handleOrientation(event) {
    const beta = event.beta;  // X축 회전 (-180 ~ 180)
    const gamma = event.gamma; // Y축 회전 (-90 ~ 90)
    
    // 초기 각도 캘리브레이션 (처음 한 번만)
    if (!calibrated) {
      initialGamma = gamma;
      initialBeta = beta;
      calibrated = true;
      return; // 첫 프레임은 스킵
    }
    
    // 초기 각도 대비 변화량 계산
    let deltaGamma = gamma - initialGamma;
    let deltaBeta = beta - initialBeta;
    
    // Deadzone: 작은 움직임 무시 (±2도 이내)
    const deadzone = 2;
    if (Math.abs(deltaGamma) < deadzone) deltaGamma = 0;
    if (Math.abs(deltaBeta) < deadzone) deltaBeta = 0;
    
    // ±30도 범위로 제한하고 감도 적용
    const rotateY = Math.max(-30, Math.min(30, deltaGamma * 0.8));
    const rotateX = Math.max(-20, Math.min(20, deltaBeta * 0.5));
    
    img.style.transform = `rotateY(${rotateY}deg) rotateX(${rotateX}deg)`;
  }
})();
</script>
