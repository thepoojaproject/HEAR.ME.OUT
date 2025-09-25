
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hear Me Out</title>
<style>
  body {
    margin: 0;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: #0f1724;
    font-family: sans-serif;
    gap: 20px;
  }
  .progress-container {
    width: 90%;
    max-width: 500px;
    height: 4px;
    background: #1e293b;
    border-radius: 2px;
    margin-top: -15px;
    overflow: hidden;
  }
  .progress-bar {
    height: 100%;
    width: 0%;
    background: #3b82f6;
    transition: width 0.1s linear;
  }
  input {
    width: 90%;
    max-width: 500px;
    padding: 14px 16px;
    border-radius: 12px;
    border: none;
    outline: none;
    font-size: 16px;
    background: #1e293b;
    color: #fff;
  }
  input::placeholder {
    color: #94a3b8;
  }
  iframe {
    display: none;
  }
</style>
<script src="https://www.youtube.com/iframe_api"></script>
</head>
<body>

<input id="ytInput" type="text" placeholder="Pssst, Idhar!" style="text-align: left;" />
<div class="progress-container">
  <div class="progress-bar" id="progressBar"></div>
</div>
<div id="playerContainer"></div>

<script>
let ytPlayer = null;

function extractID(url) {
  if(!url) return null;
  // Only ID
  if(/^[a-zA-Z0-9_-]{11}$/.test(url.trim())) return url.trim();
  try {
    const u = new URL(url);
    if(u.hostname.includes('youtu.be')) return u.pathname.slice(1);
    if(u.searchParams.get('v')) return u.searchParams.get('v');
    const parts = u.pathname.split('/');
    const idx = parts.findIndex(p => p === 'embed');
    if(idx >= 0 && parts[idx+1]) return parts[idx+1];
  } catch(e) {
    const m = url.match(/(?:v=|\/embed\/|\.be\/)([a-zA-Z0-9_-]{11})/);
    if(m && m[1]) return m[1];
  }
  return null;
}

// Create YouTube iframe
function createPlayer(id) {
  const container = document.getElementById('playerContainer');
  container.innerHTML = '';
  const iframe = document.createElement('iframe');
  iframe.src = `https://www.youtube.com/embed/${id}?autoplay=1&controls=0&modestbranding=1&rel=0&playsinline=1&enablejsapi=1&mute=1`;
  iframe.allow = "autoplay; encrypted-media";
  iframe.style.display = "none"; // hidden
  container.appendChild(iframe);
  
  // Initialize YouTube Player API
  ytPlayer = new YT.Player(iframe, {
    events: {
      'onReady': onPlayerReady,
      'onStateChange': onPlayerStateChange
    }
  });
  
  // Try to unmute after short delay (user gesture required for audible autoplay)
  setTimeout(()=>{ try{ iframe.contentWindow.postMessage('{"event":"command","func":"unMute","args":""}','*'); }catch(e){} }, 500);
}

// YouTube Player API callbacks
function onPlayerReady(event) {
  event.target.playVideo();
  document.getElementById('ytInput').value = ''; // Clear search bar when song starts
}

function onPlayerStateChange(event) {
  if(event.data === YT.PlayerState.ENDED) {
    document.getElementById('progressBar').style.width = '100%';
  }
}

// Update progress bar based on player time
function updateProgressBar() {
  if(ytPlayer) {
    ytPlayer.getCurrentTime().then(time => {
      ytPlayer.getDuration().then(duration => {
        const progress = (time / duration) * 100;
        document.getElementById('progressBar').style.width = `${progress}%`;
      });
    });
  }
  requestAnimationFrame(updateProgressBar);
}

document.getElementById('ytInput').addEventListener('input', e=>{
  const id = extractID(e.target.value.trim());
  if(id) {
    createPlayer(id);
    e.target.value = ''; // Clear input after pasting
    e.target.blur(); // Remove focus after pasting
    // Start progress bar updates
    requestAnimationFrame(updateProgressBar);
  }
});

</script>

<footer style="position: fixed; bottom: 10px; width: 100%; text-align: center; color: #94a3b8; font-size: 14px;">
  Made with  ❤️ - By Armeen
</footer>

</body>
</html># HEAR.ME.OUT
