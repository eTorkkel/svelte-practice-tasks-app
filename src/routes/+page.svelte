<script lang="ts">
    import TasksForm from "$lib/tasks-form.svelte";
    import "@picocss/pico"
    import type { Task } from "../types";
    import type { Filter } from "../types";
	import TasksList from "$lib/tasks-list.svelte";

    //Header message
    let message = "Tasks app";

    //Declare the current filter, options imported from types.ts
    let currentFilter = $state<Filter>("all");

    //Declare the tasks as an array
    let tasks = $state<Task[]>([]);

    //Calculate total tasks done
    let totalDone = $derived(
        tasks.reduce(
            (total, task) => total + Number(task.done),
            0
        )
    );

    //Derive filtered tasks from all tasks by currentfilter
    let filteredTasks = $derived.by(() => {
        switch(currentFilter) {
            case "all": {
                return tasks;
            }
            case "done": {
                return tasks.filter((task) => task.done);
            }
            case "todo": {
                return tasks.filter((task) => !task.done);
            }
            
            }
        }
    );

    //Add a new task
    function addTask(newTask: string) {
        tasks.push({
            id: crypto.randomUUID(),
            title: newTask,
            done: false,
        })
    }
    //Toggle the task done state
    function toggleDone(task:Task) {
        task.done = !task.done;
    }
    //Remove a task
    function removeTask(id: string) {
        const index = tasks.findIndex(
            (task) => task.id === id
        );
        tasks.splice(index, 1);
    }

</script>

{#snippet filterButton(filter: Filter)}
    <button
        onclick={() => currentFilter = filter}
        class:contrast={currentFilter === filter}
        class="secondary filterButton">{filter}</button
        >
{/snippet}
<main>
    <h1>{message}</h1>
    <TasksForm {addTask} />
    <p>
        {#if tasks.length}
            {totalDone} / {tasks.length} tasks completed
        {:else}
            Add a task to get started
        {/if}
    </p>
    {#if tasks.length}
        <div class="button-container">
            {@render filterButton("all")}
            {@render filterButton("todo")}
            {@render filterButton("done")}
        </div>
    {/if}
    <TasksList tasks={ filteredTasks } {toggleDone} {removeTask}/>
</main>


<style>
    main {
        margin: 1rem auto;
        color: #ffffff;
        max-width: 800px;
    }
    .button-container {
        display: flex;
        justify-content: end;
        margin-bottom: 1rem;
        gap: 0.5rem;
    }
    .filterButton {
        text-transform: capitalize;
    }
</style>