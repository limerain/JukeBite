<style>
section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  flex: 0.6;
}

h1 {
  width: 100%;
}

.welcome {
  display: block;
  position: relative;
  width: 100%;
  height: 0;
  padding: 0 0 calc(100% * 495 / 2048) 0;
}

.welcome img {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  display: block;
}

#player {
  position: relative;
  width: 100%;
  height: 0;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
}

#player iframe {
  display: contents;
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>

<script lang="ts">
// import Counter from './Counter.svelte';
// import welcome from '$lib/images/svelte-welcome.webp';
// import welcome_fallback from '$lib/images/svelte-welcome.png';
import { onMount } from 'svelte';
interface Video {
  id: string;
  start: number;
  played: boolean;
}

let currentVideoTitle: string = 'no queue';
// Initialize the queue
let queue: Video[] = [];

// Create the player and start playback
let player: any;
function onYouTubeIframeAPIReady() {
  player = new YT.Player('player', {
    height: '360',
    width: '640',
    // videoId: 'M7lc1UVf-VE',
    playerVars: {
      rel: 0, //연관동영상 표시여부(0:표시안함)
      controls: 0, //플레이어 컨트롤러 표시여부(0:표시안함)
      autoplay: 1, //자동재생 여부(1:자동재생 함, mute와 함께 설정)
      mute: 0, //음소거여부(1:음소거 함)
      loop: 0, //반복재생여부(1:반복재생 함)
      playsinline: 1, //iOS환경에서 전체화면으로 재생하지 않게
    },
    events: {
      onReady: onPlayerReady,
      onStateChange: onPlayerStateChange,
    },
  });
}

// Load the YouTube iFrame API asynchronously
onMount(() => {
  const tag = document.createElement('script');
  tag.src = 'https://www.youtube.com/iframe_api';
  const scriptTag = document.getElementsByTagName('script');
  const firstScriptTag = scriptTag[0];
  if (firstScriptTag) {
    firstScriptTag.parentNode?.insertBefore(tag, firstScriptTag);
  } else {
    const body = document.getElementsByTagName('body')[0];
    body.appendChild(tag);
  }
  window.onYouTubeIframeAPIReady = onYouTubeIframeAPIReady;
});

function playVideo() {
  const video = queue[0];
  player.loadVideoById(video.id, video.start);
  video.played = true;
}

// Start playing the first video in the queue when the player is ready
function onPlayerReady() {
  if (queue.length > 0) {
    // player.loadVideoById(queue[0].id);
    // queue[0].played = true;

    playVideo();
  }
}

// Play the next video in the queue when the current one ends
function onPlayerStateChange(event: any) {
  if (event.data == YT.PlayerState.PLAYING) {
    currentVideoTitle = player.getVideoData().title;
    queue.shift();
  }
  if (event.data == YT.PlayerState.ENDED) {
    queue[0].played = true;
    queue.shift();
    if (queue.length > 0) {
      player.loadVideoById(queue[0].id);
      queue[0].played = true;
    }
  }
}

function addVideoToQueue() {
  const input = document.getElementById('youtube-link-input') as HTMLInputElement;
  const videoData = getVideoIdFromLink(input.value);
  if (videoData !== null) {
    const { id, start } = videoData;
    queue.push({ id, start, played: false });
    input.value = '';

    // If there are no videos playing, start playing the first video in the queue
    if (queue.length === 1 && player.getPlayerState() !== YT.PlayerState.PLAYING) {
      //   player.loadVideoById(queue[0].id);
      //   queue[0].played = true;

      playVideo();
    }
  }
}

// Extract the video ID from a YouTube link
function getVideoIdFromLink(link: string): { id: string; start: number } | null {
  const match = link.match(/(?:youtube\.com\/watch\?v=|youtu\.be\/)([a-zA-Z0-9_-]{11})(?:\S+)?(?:[?&]t=(\d+)s)?/);
  if (match !== null) {
    const startTimeMatch = link.match(/t=(\d+)/);
    const startTime = startTimeMatch ? parseInt(startTimeMatch[1]) : 0;
    return {
      id: match[1],
      start: startTime,
    };
  }
  return null;
}
</script>

<svelte:head>
  <title>JukeBite: YouTube Audio Player</title>
  <meta name="description" content="Svelte demo app" />
</svelte:head>

<section>
  <h1>Current Music: {currentVideoTitle}</h1>
  <div id="player"></div>
  <input type="text" id="youtube-link-input" placeholder="Enter YouTube URL here" />
  <button on:click="{addVideoToQueue}">Add to Queue</button>
  {#if queue.length > 0}
    <h2>Queue:</h2>
    <ul>
      {#each queue as video}
        <li>{video.id}</li>
      {/each}
    </ul>
  {/if}
  <!-- {#if queue.length > 0}
    <h2>Queue:</h2>
    <ul>
      {#each queue as video}
        <li class:playing="{video.id === player.getVideoData().video_id}">
          {video.id}
        </li>
      {/each}
    </ul>
  {/if} -->
</section>
