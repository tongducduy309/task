<script>
  import Login from './Login.svelte';
  import { onMount } from 'svelte';
  import { username, user } from './user';
  import debounce from 'lodash.debounce';

  import GUN from 'gun';
  const db = GUN();

  let newTask = {};
  let tasks = [];

  let scrollBottom;
  let lastScrollTop;
  let canAutoScroll = true;
  let openFormAddTask = false;
  const key = '#foo123';

  function deleteTask(id){
    tasks = tasks.filter((task)=>task.id!=id)
    db.get('tasks_1').get(id).put(null)
  }


  function autoScroll() {
    setTimeout(() => scrollBottom?.scrollIntoView({ behavior: 'auto' }), 50);
  }

  function watchScroll(e) {
    canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
    lastScrollTop = e.target.scrollTop;
  }

  $: debouncedWatchScroll = debounce(watchScroll, 1000);

  onMount(() => {
    var match = {
      // lexical queries are kind of like a limited RegEx or Glob.
      '.': {
        // property selector
        '>': new Date(+new Date() - 1 * 1000 * 60 * 60 * 1000).toISOString(), // find any indexed property  hours ago
      },
      '-': 1, // filter in reverse
    };
    tasks = [];

    // Get Task
    db.get('tasks_1')
      .map(match)
      .once(async (data, id) => {
        if (data) {
          var task = {
            // transform the data
            who: await db.user(data).get('alias'),
            what: JSON.parse((await SEA.decrypt(data.what, key)+"").replace("#","")), 
            when: GUN.state.is(data, 'what'), 
            id:id
          };
          console.log(task);

          if (task.what) {
            tasks = [...tasks, task].sort((a, b) => a.when - b.when);
            if (canAutoScroll) {
              autoScroll();
            } 
          }
        }
      });
  });

  async function sendTask() {
    const date = newTask.deadline.split("-")
    newTask.deadline = `${date[2]}/${date[1]}/${date[0]}`
    const json = JSON.stringify(newTask);
    const secret = await SEA.encrypt("#"+json.toString(), key);
    console.log(secret);
    const task = user.get('all').set({ what: secret});
    console.log(user);
    console.log(user.get('all'));
    const index = new Date().toISOString();
    db.get('tasks_1').get(index).put(task);
    newTask = {};
    openFormAddTask = false;
    canAutoScroll = true;
    autoScroll();
  }

  function clickBtnNewTask(){
  openFormAddTask=!openFormAddTask
}
</script>



<div class="container_">
  {#if $username}
<main on:scroll={debouncedWatchScroll}>
  {#each tasks as task (task.when)}
  <div class={`task`}>
    <div class="header-task">
      <!-- {#if !task.what.finish}
      <span class="complete" on:click={()=>completeTask(task)}>Hoàn thành</span>
  
      {/if} -->
      
      <strong>{task.what.executor}</strong>
      
      {#if $username===task.who}
      <span class="delete" on:click={() => deleteTask(task.id)}>Xóa</span>
  
      {/if}
    </div>
    <div class="content-task">
      <span>Công việc: {task.what.task}</span>
      <span>Thời hạn: {task.what.deadline?task.what.deadline:"--"}</span>
      <!-- <span style="margin-bottom: 10px;">Ngày hoàn thành: {task.what.finish?task.what.finish:"--"}</span> -->
  
      <div class="time">
        <time>Người gửi: <strong>{task.who}</strong>   {new Date(task.when).toLocaleTimeString()} {new Date(task.when).getDate()+"/"+new Date(task.when).getMonth()+"/"+new Date(task.when).getFullYear()}</time>
      </div>
    </div>
  </div>
  {/each}

  <div class="dummy" bind:this={scrollBottom} />
</main>
{:else}
      <Login />
  {/if}
</div>
{#if openFormAddTask}
<form on:submit|preventDefault={sendTask} >
  <input type="text" placeholder="Người thực thi" bind:value={newTask.executor} maxlength="100" />
  <textarea placeholder="Công việc" bind:value={newTask.task}></textarea>
  <input type="date" placeholder="Ngày bắt đầu" bind:value={newTask.deadline} maxlength="100" />
  <!-- <input type="date" placeholder="Ngày hoàn thành" bind:value={newTask.finish} maxlength="100" /> -->
  <button type="submit">Giao</button>
</form>
   {/if}


<div class="btn-new-task" on:click={clickBtnNewTask}>
  <span>+</span>
</div>
