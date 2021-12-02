<script>
  import { Router, Link, Route } from 'svelte-routing';
  import { getContext } from 'svelte';
  import { ROUTER } from 'svelte-routing/src/contexts';

  const providers = ['twitter', 'github', 'aad'];
  const redirect = window.location.pathname;

  const { activeRoute } = getContext(ROUTER);

  function getProps({ location, href, isPartiallyCurrent, isCurrent }) {
    const isActive = href === '/' ? isCurrent : isPartiallyCurrent || isCurrent;

    // The object returned here is spread on the anchor element's attributes
    if (isActive) {
      return { class: 'router-link-active' };
    }
    return {};
  }

  import { onMount } from 'svelte';

let userInfo = undefined;

onMount(async () => (userInfo = await getUserInfo()));

async function getUserInfo() {
  try {
    const response = await fetch('/.auth/me');
    const payload = await response.json();
    const { clientPrincipal } = payload;
    return clientPrincipal;
  } catch (error) {
    console.error('No profile could be found');
    return undefined;
  }
}
</script>

<nav class="column is-2 menu">
  <p class="menu-label">Menu</p>
  <ul class="menu-list">
    <Link to="/products" {getProps}>Products</Link>
    <Link to="/about" {getProps}>About</Link>
  </ul>

  <nav class="menu auth">
    <p class="menu-label">Auth</p>
    <div class="menu-list auth">
      {#if !userInfo}
        {#each providers as provider (provider)}
          <a href={`/.auth/login/${provider}?post_login_redirect_uri=${redirect}`}>
            {provider}
          </a>
        {/each}
      {/if}
      {#if userInfo}
        <a href={`/.auth/logout?post_logout_redirect_uri=${redirect}`}>
          Logout
        </a>
      {/if}
    </div>
  </nav>

</nav>

{#if userInfo}
<div class="user">
  <p>Welcome</p>
  <p>{userInfo && userInfo.userDetails}</p>
  <p>{userInfo && userInfo.identityProvider}</p>
</div>
{/if}
