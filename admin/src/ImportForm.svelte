<script>
  let loading = false;
  let preview_data = undefined;
  let view_preview = false;
  let uploadUrl = "http://localhost:3280/api/v1/web/guest/util/upload";

  import { formData } from "./store";
  export let form = {};
  export let url;

  async function parseForm() {
    let submit = await fetch(uploadUrl, {
      method: "POST",
      body: new FormData(document["_theForm_"])
    })
    let data = await submit.json()
    return data
  }

  async function importer() {
    // gather data from form
    let data = await parseForm();
    preview_data = data;
    // perform import
    if(data && !view_preview) {
      confirm();
    }
    else if (view_preview && preview_data){
      console.log('show preview');
    }
    else error = "cannot parse form";
  }

  async function confirm(){
    loading = true;
    fetch(url, {
      method: "POST",
      body: JSON.stringify(preview_data),
      headers: {
        "Content-Type": "application/json"
      }
    }).then(
      async function(res) {
        let data = await res.json();
        formData.set(data);
        loading = false;
      },
      function(err) {
        error = err.message;
        formData.set({ error: err.message });
        loading = false;
      }
    );
  }


</script>

<form name="_theForm_" action={uploadUrl} method="post" enctype="multipart/form-data">
  {#each form as field}
    {#if field.type == 'message'}
      <div class="callout">
        <div class="callout-title">{field.name}</div>
        <p>{field.description}</p>
      </div>
      <br>
    {:else if field.type == 'string'}
      <div class="form-group">
        <input
          type="text"
          class="form-control"
          id={field.name}
          name={field.name}
          bind:value={field.value} />
        <label class="active" for={field.name}>{field.description}</label>
      </div>
    {:else if field.type == 'password'}
      <div class="form-group">
        <label class="active" for={field.name}>{field.description}</label>
        <input
          type="password"
          class="form-control"
          id={field.name}
          name={field.name}
          bind:value={field.value} />
      </div>
    {:else if field.type == 'upload'}
      <div class="form-group">
        <input
          type="file"
          class="form-control"
          id={field.name}
          name={field.name}
          bind:files={field.value} />
        <label class="active" for={field.name}>{field.description}</label>
      </div>
    {:else if field.type == 'textarea'}
      <div class="form-group">
        <textarea 
        id={field.name}
        name={field.name}
        rows="3" 
        bind:value={field.value} />
        <label for={field.name}>{field.description}</label>
      </div>
      <br />
    {:else}
      <div class="alert alert-danger" role="alert">
        Error: Unknown or empty type {field.type}
      </div>
    {/if}
  {:else}
    <div class="alert alert-danger" role="alert">Error: no form data</div>
  {/each}
  <div>
    {#if loading}
      <div>loading...</div>
    {:else}
      <input
        id="view_preview"
        type="checkbox"
        bind:checked={view_preview}
      />
      <label for="view_preview">
        View preview before importing data
      </label>
      <button type="button" class="btn btn-primary" on:click={importer}>
        {#if view_preview}Preview data{:else}Import{/if}
      </button>
    {/if}

    {#if view_preview && preview_data}
      <button type="button" class="btn btn-primary" on:click={confirm}>
        Import
      </button>
    {/if}
  </div>
</form>
