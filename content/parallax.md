---
title: "Parallax"
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
    
    // 값을 적절히 스케일링
    const rotateY = Math.max(-60, Math.min(60, gamma * 1.0));
    const rotateX = Math.max(-40, Math.min(40, (beta - 90) * 0.6));
    
    img.style.transform = `rotateY(${rotateY}deg) rotateX(${rotateX}deg)`;
  }
})();
</script>
