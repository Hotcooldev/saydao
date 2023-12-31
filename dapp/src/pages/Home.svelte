<script>
  import { wallet } from "src/state/eth";
  import { get as getMember, totalMembers, me } from "src/state/dao/member";
  import { upcomingMeetings } from "src/state/dao/poll";
  import { role, memberId } from "src/state/dao";
  import Banner from "src/components/banner";
  import Meetings from "src/components/meeting";
  import CONFIG from "src/config";
  import db from "../state/db";

  window.me = me;
  let hideMagicWordsBanner = db.get("saydao:hideMagicWordsBanner");

  function handleDismissBanner() {
    if (
      confirm(
        "I understand that I cannot login again if I don't have my magic words."
      )
    ) {
      db.set("saydao:hideMagicWordsBanner", true);
      hideMagicWordsBanner = true;
    }
  }
</script>

<style>
  .participate {
    padding: var(--size-l) 0;
  }

  .participate .body {
    display: flex;
    flex-direction: var(--flex-direction);
  }

  .save-mnemonic {
    border: 1px solid black;
    border-radius: var(--size-xxs);
    padding: var(--size-m) var(--size-s);
    background-color: var(--join--color-working);
    margin-bottom: var(--size-l);
  }

  .save-mnemonic h2 {
    margin-top: 0;
  }
</style>

<Banner />

{#if $role.member && $wallet.mnemonic && !hideMagicWordsBanner}
  <section class="save-mnemonic">
    <h2>Important!</h2>
    <p>
      Make sure to save your
      <strong>12 magic words</strong> before you leave this page. You need these
      magic words to log in again. Write them down and keep them safe.
    </p>
    <a class="button" href="#/settings">Save your magic words</a>
    <button on:click={handleDismissBanner}> Dismiss banner </button>
  </section>
{/if}

<Meetings
  list={$upcomingMeetings && $upcomingMeetings.length && [$upcomingMeetings[0]]}
  highlightFirst={true}
/>

<div class="participate">
  <section>
    <h2>Participate</h2>
  </section>
  <section class="body">
    <div>
      <h3>Events</h3>
      <ul>
        <li>
          <a href="#/events/upcoming">Upcoming events</a>: Show upcoming {CONFIG.name}
          events.
        </li>
        <li>
          <a href="#/events/past">Past events</a>: Show past {CONFIG.name} events.
        </li>
        {#if $role.member}
          <li>
            <a href="#/events/create">Propose an event</a>: Propose a new {CONFIG.name}
            event.
          </li>
        {/if}
      </ul>
    </div>
    <div>
      <h3>Polls</h3>
      <ul>
        <li>
          <a href="#/polls/open">Open polls</a>: Show polls you can vote on.
        </li>
        <li>
          <a href="#/polls/closed">Closed polls</a>: Show polls that are now
          closed.
        </li>
        {#if $role.member}
          <li>
            <a href="#/polls/create">New poll</a>: Open a poll for {CONFIG.name}
            to vote on.
          </li>
        {/if}
      </ul>
    </div>
  </section>
</div>

<section>
  {#if $role.admin || $role.manager}
    <div>
      <h2>Administration</h2>
      <p>
        You are an <strong>admin for {CONFIG.name}</strong>. You have special
        powers. Use them wisely.
      </p>
      <ul>
        <li>
          <a href="#/member/invite">Add a member to {CONFIG.name}</a>.
        </li>
      </ul>
    </div>
  {/if}

  <div>
    <h2>Members</h2>
    <p>There are currently {$totalMembers} members of {CONFIG.name}.</p>
    <ul>
      <li>
        <a href="#/member/list">List members.</a>
      </li>
    </ul>
  </div>
</section>

<section>
  <div>
    <h2>What is {CONFIG.name}</h2>
    <p>
      {CONFIG.name} is an experiment in facilitating group decision-making in loosely
      defined communities. Find out more:
    </p>
    <ul>
      <li>
        <a href="#/docs/operation.md">See how {CONFIG.name} works.</a>
      </li>
      <li>
        <a href="#/docs/principles.md">Read our design principles.</a>
      </li>
    </ul>
  </div>
</section>
