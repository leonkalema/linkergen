<!-- src/LinkRedirector.svelte -->
<script>
  import { onMount } from 'svelte';
  import { writable, derived } from 'svelte/store';

  let websiteUrl = '';
  let appleStoreUrl = '';
  let androidStoreUrl = '';
   let generatedUrl = '';
  const error = writable('');

  const showError = derived(error, ($error) => $error !== '');

  const isValidUrl = (url) => {
    try {
      new URL(url);
      return true;
    } catch (_) {
      return false;
    }
  };


const copyToClipboard = async () => {
    try {
      await navigator.clipboard.writeText(generatedUrl);
      alert('URL copied to clipboard');
    } catch (err) {
      console.error('Failed to copy URL: ', err);
      alert('Failed to copy URL. Please try again.');
    }
  };


  const handleSubmit = async (event) => {
    event.preventDefault();
    error.set('');

    if (!isValidUrl(websiteUrl) || !isValidUrl(appleStoreUrl) || !isValidUrl(androidStoreUrl)) {
      error.set('Please provide valid URLs for all fields.');
      return;
    }

    const payload = { websiteUrl, appleStoreUrl, androidStoreUrl };
    const response = await fetch('https://shy-tan-basket-clam-veil.cyclic.app/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });

    if (response.ok) {
      const data = await response.json();
      generatedUrl = data.generatedUrl;
      console.log('Generated URL:', generatedUrl);
    } else {
      error.set('Error generating URL. Please try again.');
    }
  };
</script>

<style>

  /* Google-inspired Material Design styles */
  body {
    font-family: 'Roboto', sans-serif;
  }

  form {
    background-color: white;
    max-width: 500px;
    margin: 40px auto;
    padding: 32px;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
  }

  label {
    display: block;
    font-size: 16px;
    font-weight: 500;
    margin-bottom: 8px;
  }

  input[type='url'] {
    width: 100%;
    font-size: 16px;
    padding: 8px;
    margin-bottom: 16px;
    border: none;
    border-bottom: 2px solid #4a4a4a;
    outline: none;
    transition: border-color 0.3s;
  }

  input[type='url']:focus {
    border-bottom-color: #1a73e8;
  }

  button {
    background-color: #1a73e8;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 8px 16px;
    font-size: 16px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  button:hover {
    background-color: #1565c0;
  }

  p {
    color: #d32f2f;
    font-weight: 500;
  }
</style>

<form on:submit={handleSubmit}>
  <div>
    <label for="websiteUrl">Website URL:</label>
    <input type="url" id="websiteUrl" bind:value={websiteUrl} required />
  </div>
  <div>
    <label for="appleStoreUrl">Apple Store URL:</label>
    <input type="url" id="appleStoreUrl" bind:value={appleStoreUrl} required />
  </div>
  <div>
    <label for="androidStoreUrl">Android Store URL:</label>
    <input type="url" id="androidStoreUrl" bind:value={androidStoreUrl} required />
  </div>
  <button type="submit">Submit</button>
  {#if $showError}
    <p>{$error}</p>
  {/if}
</form>


{#if generatedUrl}
  <div>
    <h3>Generated URL:</h3>
      <a href="{generatedUrl}" target="_blank" rel="noopener noreferrer">{generatedUrl}</a>
    <button on:click={copyToClipboard}>Copy Link</button>
  </div>
{/if}
