<script>
  import { username, user } from './user';

  function signout() {
    user.leave();
    username.set('');
  }
</script>

<header>
  {#if $username}
    <div class="user-bio" style="color:#000">
      
      <span>Hello <strong >{$username}</strong></span>
    </div>

    <button class="signout-button" on:click={signout}>Sign Out</button>

  {/if}
</header>