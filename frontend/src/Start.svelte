<svelte:window
  on:keydown={(e) => {
    $ctrlKey = e.ctrlKey;
    $shiftKey = e.shiftKey;
    $metaKey = e.metaKey;
    if(($skipKey && (e.key === 'Enter'))||(showComponent !== 'filemanager')) {
      $keyProcess = true;
    } else {
      if($keyProcess) {
        if($processKey !== null) $processKey(e);
      }
    }
    $skipKey = false;
  }}
  on:keyup={(e) => {
    $ctrlKey = e.ctrlKey;
    $shiftKey = e.shiftKey;
    $metaKey = e.metaKey;
  }}
  on:resize={(e) => {
    midSize = window.innerHeight - 68;
  }}
/>

<div
  id='bodyContainer'
>

  <TitleBar />

  <FileManager 
    on:switchView={switchView}
    mid={midSize}
  />

  <StatusLine />
</div>

{#if showComponent === 'preferences'}
  <Preferences
    on:switchView={switchView}
  />
{/if}

<style>
  :global(body) {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 100vh;
    user-select: none;
    overflow: hidden;
  }

  #bodyContainer {
    display: flex;
    flex-direction: column;
    height: 100vh;
    width: 100%;
    margin: 0px;
    padding: 0px;
    min-height: 100vh;
    min-width: 100%;
  }
</style>

<script>
  import { onMount } from "svelte";
  import FileManager from "./components/FileManager.svelte";
  import Preferences from "./components/Preferences.svelte";
  import StatusLine from './components/StatusLine.svelte';
  import TitleBar from './components/TitleBar.svelte';
  import { shiftKey } from "./stores/shiftKey.js";
  import { ctrlKey } from "./stores/ctrlKey.js";
  import { metaKey } from "./stores/metaKey.js";
  import { altKey } from "./stores/altKey.js";
  import { skipKey } from "./stores/skipKey.js";
  import { processKey } from "./stores/processKey.js";
  import { keyProcess } from './stores/keyProcess.js';

  let showComponent = 'filemanager';
  let midSize = null;

  onMount(() => {
    midSize = window.innerHeight - 68;
  });

  function switchView(v) {
    showComponent = v.detail.view;
  }
</script>
