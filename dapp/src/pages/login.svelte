<script>
  import { push } from "svelte-spa-router";
  import { login } from "src/state/eth";
  import etherea from "etherea";

  let mnemonic = "";
  let error;

  async function submitMnemonic() {
    error = false;
    try {
      await login(mnemonic.trim());
    } catch (e) {
      console.log(e);
      error = e.toString();
      throw e;
    }
    push("/");
  }

  async function handleLogin() {
    error = false;
    try {
      await login();
    } catch (e) {
      console.log(e);
      error = e.toString();
      throw e;
    }
    push("/");
  }
</script>

<section>
  <div>
    <h2>Log in</h2>

    {#if etherea.hasNativeWallet()}
      <p>Your browser supports Ethereum.</p>

      <button class="button-shadow" on:click={handleLogin} href="#"
        ><span>Log in with your Ethereum account</span></button
      >
      {#if error}
        <p class="error">It didn't work out. Try again please.</p>
        <details>
          {error}
        </details>
      {/if}
    {:else}
      <p>Please enter your 12 magic words.</p>

      <fieldset>
        <legend> Login </legend>
        <form on:submit|preventDefault={submitMnemonic}>
          {#if error}
            <p class="error">
              It didn't work out. Please double check your 12 magic words.
            </p>
            <details>
              {error}
            </details>
          {/if}
          <textarea
            required
            placeholder="example words friend ... ... ... ... ... ... ... ... ..."
            bind:value={mnemonic}
          />
          <button type="submit">Log in</button>
        </form>
      </fieldset>
      <a href="mailto:agranzot@mailbox.org?subject=I forgot my 12 magic words"
        >Recover your account</a
      >.
    {/if}
  </div>
</section>
