<script>
    import { currentTasks } from "$lib/store";
    import { createTask, deleteTaskById, updateTaskById } from "$lib/utils/tasks";


    // get all fields in cat and group by cat in an array
    $: category = [...new Set($currentTasks.map(item => item.cat))];
    // get all fields in project and group by project in an array
    $: project = [...new Set($currentTasks.map(item => item.project))];
    

    function startTask(task) {
        task.start = new Date();

        const data = {
            start: task.start,
        }
        updateTaskById(task.id, data);
        
    }

    function endTask(task) {
        task.end = new Date();

        const data = {
            end: task.end,
        }
        updateTaskById(task.id, data);
    }

    function updateTaskName(task) {
        const data = {
            name: task.name,
        }
        updateTaskById(task.id, data);
    }

    function updateTaskCat(task) {
        const data = {
            cat: task.cat,
        }
        updateTaskById(task.id, data);
    }

    function updateTaskProject(task) {
        const data = {
            project: task.project,
        }
        updateTaskById(task.id, data);
    }

    // check the time passed since start
    function checkPassedTime(task) {
        const now = new Date();

        // convert task.start to date
        const start = new Date(task.start);
        const diff = now - start;
        const seconds = Math.floor(diff / 1000);
        const minutes = Math.floor(seconds / 60);
        const hours = Math.floor(minutes / 60);
        const days = Math.floor(hours / 24);

        return {
            days,
            hours: hours % 24,
            minutes: minutes % 60,
            seconds: seconds % 60,
        };
    }

    // Check the total time passed between start and end
    function checkTotalTime(task) {
        // convert task.start to date
        const start = new Date(task.start);
        // convert task.end to date
        const end = new Date(task.end);
        const diff = end - start;
        const seconds = Math.floor(diff / 1000);
        const minutes = Math.floor(seconds / 60);
        const hours = Math.floor(minutes / 60);
        const days = Math.floor(hours / 24);

        // Return a string hh:mm
        return `${hours}h ${minutes}m`;
    }

    // format date to dd.mm.yy - hh:mm
    function compactDate(date) {
        if (!date) return "";
        // only display and change format dd:mm:yy - hh:mm
        const d = new Date(date);
        // slice year to last 2 digits
        const year = String(d.getFullYear()).slice(-2);
        const month = String(d.getMonth() + 1).padStart(2, '0');
        const day = String(d.getDate()).padStart(2, '0');
        const hour = String(d.getHours()).padStart(2, '0');
        const minute = String(d.getMinutes()).padStart(2, '0');
        return `${day}.${month}.${year} - ${hour}:${minute}`;

    }

    // A clock that updates every second
    setInterval(() => {
        $currentTasks.forEach(task => {
            if (task.start && !task.end) {
                const time = checkPassedTime(task);
                task.timer = time
            }
        });
        $currentTasks = $currentTasks;
    }, 1000);
    

</script>
<div>
    <h1 class="text-3xl font-bold">Time Tracking</h1>
    <div>
        <button class="btn" on:click={createTask}> 📝 New Task</button>
    </div>
    <div class="overflow-x-auto">
        <table class="table table-compact w-full">
          <!-- head -->
          <thead>
            <tr>
              <th>name</th>
              <th>start</th>
              <th>end</th>
              <th>category</th>
              <th>project</th>
              <th>timer</th>
              <th>trigger</th>
              <th>Delete</th>
            </tr>
          </thead>
          <tbody>
            {#each $currentTasks as task}
                <tr>
                    <td><input type="text" bind:value={task.name} on:change={() => { updateTaskName(task) }} /></td>
                
                    <td>{compactDate(task.start)}</td>
                    <td>{compactDate(task.end)}</td>
                    <td>
                        <input type="text" bind:value={task.cat} on:change={() => { updateTaskCat(task) }} />
                        <br>
                        <select bind:value={task.cat} on:change={() => { updateTaskCat(task) }} >
                        {#each category as cat}
                            <option value={cat}>{cat}</option>
                        {/each}
                        </select>
                    </td>
                    <td>
                        <input type="text" bind:value={task.project} on:change={() => { updateTaskProject(task) }} />
                        <br>
                        <select bind:value={task.project} on:change={() => { updateTaskProject(task) }} >
                        {#each project as proj}
                            <option value={proj}>{proj}</option>
                        {/each}
                        </select>
                    </td>
                    <td>
                        {#if task.timer}
                        <span class="countdown">
                            <span style="--value:{task.timer.hours};"></span>h
                            <span style="--value:{task.timer.minutes};"></span>m
                            <span style="--value:{task.timer.seconds};"></span>s
                          </span>
                        {/if}
                        {#if task.start && task.end}
                            {checkTotalTime(task)}
                        {/if}
                    </td>
                    <td>
                        {#if task.start && task.end}
                            Ended
                        
                        {:else}
                            {#if task.start}
                                <button class="btn" on:click={() => { endTask(task) }}>End</button>
                            {:else}
                                <button class="btn" on:click={() => { startTask(task) }}>Start</button>
                            {/if}
                        {/if}
                    </td>
                    <td><span class="cursor-pointer" on:click={() => { deleteTaskById(task.id) }}>🗑️</span></td>
                </tr>
            {/each}
          </tbody>
        </table>
      </div>
</div>

<style>
    input {
        width: fit-content;
        max-width: 90px;
    }
</style>