<script>
  import { wallet } from "src/state/eth";
  import { list as members } from "src/state/dao/member";
  import Loading from "src/components/Loading.svelte";
  import { createBitmaps, toBinary } from "../utils";

  export let meetingId;
  export let meetingState;
  export let onDone;

  let memberIds = [];
  let state = {};

  function handleClose() {
    state = {};
  }

  async function handleSubmit() {
    state.action = "submit";
    console.log("Submit presence for members", memberIds);
    const bitmaps = createBitmaps(memberIds);
    try {
      for (let cluster in bitmaps) {
        console.log("Cluster", cluster);
        console.log("Bitmap", toBinary(bitmaps[cluster]));
        const receipt =
          await $wallet.contracts.SayDAO.updateMeetingParticipants(
            meetingId,
            cluster,
            bitmaps[cluster]
          );
        console.log("Wait for tx", receipt.hash);
        await receipt.wait();
      }
      console.log("Seal participant list");
      const receipt = await $wallet.contracts.SayDAO.sealMeetingParticipants(
        meetingId
      );
      console.log("Wait for tx", receipt.hash);
      await receipt.wait();
      await handleTokenDistribution();
      state = {};
    } catch (e) {
      console.error(e);
      state.error = e.toString();
    }
  }

  async function handleTokenDistribution() {
    console.log("Start token distribution");

    while (true) {
      const toProcess =
        await $wallet.contracts.SayDAO.getRemainingDistributionClusters(
          meetingId
        );

      if (toProcess.isZero()) break;

      console.log("Clusters left to process", toProcess.toNumber());

      const receipt = await $wallet.contracts.SayDAO.distributeMeetingTokens(
        meetingId,
        64
      );
      console.log("Wait for tx", receipt.hash);
      await receipt.wait();
    }

    state = "idle";
    onDone();
  }
</script>

<Loading {state} onClose={handleClose} />

{#if meetingState === "initial"}
  <h2>Select the participants</h2>

  <p>
    You are the supervisor of this event. Select the members who participated in
    the event to award Say tokens.
  </p>

  <form on:submit|preventDefault={handleSubmit}>
    <formset>
      {#if $members}
        {#each $members as member}
          <label>
            <input type="checkbox" bind:group={memberIds} value={member.id} />
            Member #{member.id}
          </label>
          <br />
        {/each}
      {:else}
        Wait, loading members...
      {/if}
    </formset>

    <button type="submit">Submit</button>
  </form>
{/if}

{#if meetingState === "sealed"}
  <h2>Distribute tokens to participants</h2>

  <p>
    You are the supervisor of this event. Please distribute Say tokens to
    participants. Be patient! This might take a while if there were a lot of
    participants.
  </p>
  <form on:submit|preventDefault={handleTokenDistribution}>
    <button type="submit">Start distributing Say tokens</button>
  </form>
{/if}
