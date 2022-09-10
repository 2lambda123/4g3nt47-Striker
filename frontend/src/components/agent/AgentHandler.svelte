<script>
  
  /**
   * @file This component handles the interaction of user with an active agent.
   * @author Umar Abdul
   */

  import {createEventDispatcher} from 'svelte';
  import {scale, slide, fade} from 'svelte/transition';
  import Fa from 'svelte-fa/src/fa.svelte';
  import * as icons from '@fortawesome/free-solid-svg-icons';
  import TasksList from './TasksList.svelte';

  export let session = {};
  export let socket = null;
  export let agent = {};
  export let tasks = [];
  export let consoleMsgs = [];
  
  const dispatch = createEventDispatcher();
  let tabs = ["Info", "Tasks", "Console", "Manage"]
  let currTab = tabs[0];
  let shellCommand = "";
  let consoleText = "";
  let msgCount = 0;

  const updateConsole = (consoleMsgs) => {
    
    if (msgCount === consoleMsgs.length) // A hacky solution for an autoscroll bug when nothing changes
      return;
    msgCount = consoleMsgs.length;
    consoleText = consoleMsgs.join("\n");
    // Scroll to bottom.
    let elem = document.getElementById('console-text');
    if (elem)
      setTimeout(() => elem.scrollTop = elem.scrollHeight, 50); // Need the function to return before this code is run else the last line will not be visible.
  };

  // A reactive expression to update render new console messages.
  $: updateConsole(consoleMsgs);

  const sendShellCommand = async () => {

    if (shellCommand.trim().length === 0)
      return;
    let cmd = shellCommand;
    shellCommand = "";
    socket.emit("create_task", {
      agentID: agent.uid,
      taskType: "system",
      data: {
        cmd
      }
    });
  };

</script>

<div class="border-2 border-gray-900 min-h-full max-h-full no-scrollbar overflow-y-auto">

  <!-- Our tabs -->
  <ul class="list-none text-center bg-gray-900 text-white">
    {#each tabs as tab, index}
      <li class={`inline-block w-32 px-5 mx-10 cursor-pointer ${tab === currTab ? "text-green-300 bg-gray-300 text-gray-900" : ""}`} on:click={() => currTab = tab}>{tab}</li>
    {/each}
  </ul>

  <!-- A div to wrap all our tabs -->
  <div class="m-3 text-lg">
    <!-- Render the current tab -->
    {#if (currTab === "Info")}
      <table class="w-full border-2 border-black text-left font-mono">
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">ID:</th>
          <td>{agent.uid}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">OS:</th>
          <td>{agent.os}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">Host:</th>
          <td>{agent.host}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">User:</th>
          <td>{agent.user}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">Process ID:</th>
          <td>{agent.pid}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">Working directory:</th>
          <td>{agent.cwd}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">Date Created:</th>
          <td>{new Date(agent.dateCreated).toLocaleString()}</td>
        </tr>
        <tr class="border-b-2 border-gray-900">
          <th class="text-right pr-5">Last Contact:</th>
          <td>{new Date(agent.lastSeen).toLocaleString() + ` (${((Date.now() - agent.lastSeen) / (1000 * 60)).toFixed(2)} minutes)`}</td>
        </tr>
      </table>
    {:else if (currTab === "Console")}
      <textarea id="console-text" class="w-full no-scrollbar font-mono text-md bg-gray-900 border-2 border-black p-1 text-white break-all" rows="15" bind:value={consoleText} readonly></textarea>
      <input class="w-full border-2 border-gray-900 pl-2 font-mono bg-gray-300" type="text" placeholder="command..." bind:value={shellCommand} on:change={sendShellCommand}>
    {:else if (currTab === "Tasks")}
      <TasksList {tasks}/>
    {/if}
  </div>
</div>
