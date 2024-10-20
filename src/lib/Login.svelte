<script lang="ts">
  import {currentUser, pb} from './pocketbase';

  let username: string;
  let password: string;
  let name: string;

  async function login() {
    await pb.collection('users').authWithPassword(username, password);
  }

  async function signUp(){
    try {
      const data = {
        username,
        password,
        name,
      }

      const createdUser = await pb.collection('users').create(data);
      await login();
    } catch (err) {
      console.error(`There was an error logging you in. ${err}`);
    }

  }

  async function signOut() {
    pb.authStore.clear();
  }
</script>

{#if $currentUser}
  <p>Signed in as {$currentUser.username}</p>

{:else} 
  <form on:submit|preventDefault>
    <input
      placeholder="Username"
      type="text"
      bind:value={username}
    />
    <input
      placeholder="Password"
      type="text"
      bind:value={password}
    />
    <input
      placeholder="Name"
      type="text"
      bind:value={name}
    />
    <button on:click={signUp}>Sign Up</button>
    <button on:click={login}>Login</button>
  </form>
{/if}