<div 
  class='panel'
  bind:this={DOM}
>
  {#each entries as entry,index}
    <Entry 
      pane={pane} 
      index={index}
      entry={entry}
      utilities={utilities}
      on:changeDir={changeDir}
      on:openFile={openFile}
      on:changeViewing={changeViewingDOM}
    />
  {/each}
  <div
    class="empty"
    draggable="false"
    on:click={() => {cursorToPane()}}
    on:drop|preventDefault={(e) => { dropFiles(e, 'drop'); }}
    on:dragover|preventDefault={(e) => { dropFiles(e,'dragover'); }}
  >
  </div>
</div>

<style>
  .panel {
    display: flex;
    flex-direction: column;
    padding: 0px;
    margin: 0px;
    height: 100%;
    overflow-y: scroll;
    overflow-x: hidden;
  }

  .empty {
    height: 100%;
  }
</style>

<script>
  import { createEventDispatcher } from 'svelte';
  import { get } from 'svelte/store';
  import Entry from './Entry.svelte';
  import { currentCursor } from '../stores/currentCursor.js';
  import { currentLeftFile } from '../stores/currentLeftFile.js';
  import { currentRightFile } from '../stores/currentRightFile.js';
  import { config } from '../stores/config.js';
  import { leftDir } from '../stores/leftDir.js';
  import { rightDir } from '../stores/rightDir.js';

  export let pane = 'left';
  export let entries = [];
  export let utilities;

  const dispatch = createEventDispatcher();

  let DOM;

  function changeDir(e) {
    dispatch('changeDir', {
      dir: e.detail,
      pane: pane
    });
  }

  function openFile(e) {
    dispatch('openFile', e.detail);
  }

  function changeViewingDOM(e) {
    var elDOM = e.detail;
    if(elDOM !== null) {
      DOM.scrollTop += elDOM.dir;
      if(DOM.scrollTop < 0) DOM.scrollTop = 0;
    }
  }

  async function dropFiles(e,type) {
    var shiftKey = e.shiftKey;
    switch(type) {
      case 'over':
        if(shiftKey) {
          e.dataTransfer.dropEffect = 'move';
        } else {
          e.dataTransfer.dropEffect = 'copy';
        }
        break;
      case 'drop':
        // 
        // Get the local information for dropping into.
        //
        var dirPath = '';
        var fileName = '';
        var toEntry = {
          dir: '',
          name: '',
          fileSystemType: utilities.type,
          fileSystem: utilities,
          type: 1
        };
        if(entries.length > 0) {
          toEntry.dir = entries[0].dir;
          toEntry.name = ''
        } else {
          var curPane;
          if(pane === 'left') {
            curPane = get(leftDir);
          } else {
            curPane = get(rightDir);
          }
          toEntry.dir = curPane.path;
          toEntry.name = '';
        }

        // 
        // Process the information from the drop.
        // 
        const dataTransArray = e.dataTransfer.getData('text/plain').split('\n');
        const lconfig = get(config);
        var fromEntries = [];
        dataTransArray.forEach(async dataTrans => {
          const parts = dataTrans.split('|');
          if(parts[1] === '1') {
            dirPath = parts[0];
            var fdir = await utilities.splitFilePath(dirPath);
            const nwDir = await utilities.appendPath(toEntry.dir, fdir.name);
            await utilities.makeDir(nwDir);
            toEntry.dir = nwDir;
          } else {
            const result = await utilities.splitFilePath(parts[0]);
            dirPath = result.dir;
            fileName = result.name;
          }
          fromEntries.push({
            dir: dirPath,
            name: fileName,
            fileSystemType: utilities.type,
            fileSystem: utilities
          });
        });
        if(shiftKey) {
          lconfig.extensions.getExtCommand('moveEntriesCommand').command(fromEntries, toEntry);
        } else {
          lconfig.extensions.getExtCommand('copyEntriesCommand').command(fromEntries, toEntry);
        }
        break;
      default:
        break;
    }
  }

  function cursorToPane() {
    var nEntry = {
      dir: '',
      name: '',
      fileSystemType: utilities.type,
      fileSystem: utilities,
      type: 1,
      size: 0,
      datetime: '',
      seleted: false
    }
    if(entries.length > 0) {
      nEntry = entries[0];
    } else {
      var curPane;
      if(pane === 'left') {
        curPane = get(leftDir);
      } else {
        curPane = get(rightDir);
      }
      nEntry.dir = curPane.path;
      nEntry.name = '';
    }
    currentCursor.set({
      pane: pane,
      entry: nEntry
    });

    if(pane === 'right') {
      currentRightFile.set(
        {
          entry: nEntry
        }
      );
    } else {
      currentLeftFile.set(
        {
          entry: nEntry
        }
      )
    }
  }
</script>
