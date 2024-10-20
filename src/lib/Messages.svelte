<script lang="ts">
  import {onMount, onDestroy} from "svelte";
  import {currentUser, pb} from "./pocketbase";
  import profile from '/profile.svg';
  import type { RecordModel } from "pocketbase";

  let newMessage: String;
  let messages: RecordModel[] = [];
  let unsubscribe: () => void;

  onMount(async () => {
    const resultList = await pb.collection('messages').getList(1, 25, {
      sort: 'created',
      expand: 'user',
    });

    messages = resultList.items;

    unsubscribe = await pb
      .collection('messages')
      .subscribe("*", async ({action, record}) => {
        if (action === 'create') {
          const user = await pb.collection('users').getOne(record.user);
          record.expand = {user};

          messages = [...messages, record];
        }
        if (action === 'delete') {
          messages = messages.filter((m) => m.id !== record.id);
        }
      });
  });

  onDestroy (() => {
    unsubscribe?.();
  });

  async function sendMessage() {
    const data = {
      text: newMessage,
      user: $currentUser?.id,
    };

    const createdMessage = await pb.collection('messages').create(data);
    newMessage = "";
  }
</script>

{#if $currentUser} 
  <form on:submit|preventDefault={sendMessage}>
    <input placeholder="Message" type="text" bind:value={newMessage} />
    <button type="submit">Send</button>

  </form>
{/if}

  <div class="messages">
    {#each messages as message (message.id)}
      <div class="message">
        {#if !message.expand?.user?.avatar}
          <img
          class="avatar"
          src={profile} 
          alt="user avatar"
          width="40px"
          /> 

        {:else}
          <img
            class="avatar"
            src={`${message.expand?.user?.avatar}`} 
            alt="user avatar"
            width="40px"
          /> 
        {/if}

        <div>
          <p>
            Sent by @{message.expand?.user?.username}
          </p>
          <p class="message-text">{message.message_content}</p>
        </div>
      </div>
    {/each}
  </div>
