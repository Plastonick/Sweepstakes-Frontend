<script lang="ts">
  import Github from './lib/Github.svelte'
  import axios from 'axios'

  const sweepstakesApi = import.meta.env.VITE_SWEEPSTAKES_API

  const fetchAndHydrateConfiguration = async function () {
    if (!formValues.webhook) {
      return
    }

    const hydrateConfiguration = (result) => {
      formValues.service = result.data.service
      formValues.owners = result.data.owners
      formValues.win = result.data.win
      formValues.score = result.data.score
      formValues.kickOff = result.data.kickOff
      formValues.draw = result.data.draw
      fetchFeedback = { type: 'success', message: 'Retrieved existing configuration' }
    }

    axios.get(`${sweepstakesApi}/configuration`, { params: { url: formValues.webhook }})
      .then(hydrateConfiguration)
      .catch((result) => {
          fetchFeedback = { type: 'error', message: result.response.data.message }
      })
  }

  const hydrateService = () => {
    const isSlack = formValues.webhook.indexOf('hooks.slack.com') !== -1
    const isDiscord = formValues.webhook.indexOf('discord.com') !== -1

    if (isSlack) {
      formValues.service = 'slack'
    } else if (isDiscord) {
      formValues.service = 'discord'
    }
  }

  const testConfiguration = async function () {
      if (!formValues.webhook) {
          persistFeedback = { type: 'error', message: 'No webhook URL detected' }

          return
      }

      if (formValues.service !== 'slack' && formValues.service !== 'discord') {
          persistFeedback = {type: 'error', message: 'No valid Service detected' }

          return
      }

      await axios.post(`${sweepstakesApi}/webhook-test`, formValues)
          .then((result) => {
              persistFeedback = { type: 'success', message: result.data.message }
          })
          .catch((result) => {
              persistFeedback = { type: 'error', message: result.response.data.message }
          })
  }

  const updateConfiguration = async function () {
    if (!formValues.webhook) {
      return
    }

    await axios.put(`${sweepstakesApi}/configuration`, formValues)
        .then((result) => {
            persistFeedback = { type: 'success', message: result.data.message }
        })
        .catch((result) => {
            persistFeedback = { type: 'error', message: result.response.data.message }
        })
  }

  const deleteConfiguration = async function () {
    if (!formValues.webhook) {
      return
    }

    await axios.delete(`${sweepstakesApi}/configuration`, { params: { url: formValues.webhook }} )
        .then((result) => {
            persistFeedback = { type: 'success', message: result.data.message }
        })
        .catch((result) => {
            persistFeedback = { type: 'error', message: result.response.data.message }
        })
  }

  const formValues = {
    webhook: '',
    service: '',
    owners: {},
    win: '',
    score: '',
    kickOff: '',
    draw: '',
  }

  const teams = {
     ARG: 'Argentina',
     AUS: 'Australia',
     BEL: 'Belgium',
     BRA: 'Brazil',
     CMR: 'Cameroon',
     CAN: 'Canada',
     CRC: 'Costa Rica',
     CRO: 'Croatia',
     DEN: 'Denmark',
     ECU: 'Ecuador',
     ENG: 'England',
     FRA: 'France',
     GER: 'Germany',
     GHA: 'Ghana',
     IRN: 'Iran',
     JPN: 'Japan',
     MEX: 'Mexico',
     MAR: 'Morocco',
     NED: 'Netherlands',
     POL: 'Poland',
     POR: 'Portugal',
     QAT: 'Qatar',
     KSA: 'Saudi Arabia',
     SEN: 'Senegal',
     SRB: 'Serbia',
     KOR: 'South Korea',
     ESP: 'Spain',
     SUI: 'Switzerland',
     TUN: 'Tunisia',
     USA: 'United States',
     URU: 'Uruguay',
     WAL: 'Wales',
  }
  let fetchFeedback: { type: string, message: string }|null = null
  let persistFeedback: { type: string, message: string }|null = null

  $: if (fetchFeedback) {
      window.setTimeout(() => {
          fetchFeedback = null
      }, 5000)
  }

  $: if (persistFeedback) {
      window.setTimeout(() => {
          persistFeedback = null
      }, 5000)
  }

  const emojiTitle = 'Emoji should be a comma separated list of emoji randomly picked for their event, e.g. tada,confetti_ball,partying_face,fireworks'
  let teamsExpanded = true
  const toggleTeams = function () {
    teamsExpanded = !teamsExpanded
  }

  let helpExpanded = false
  const toggleHelp = function () {
    helpExpanded = !helpExpanded
  }
</script>

<Github/>

<div class="content">
  <h1>
    World Cup 2022 Sweepstakes Announcer
  </h1>

  <h5>
    Add an announcer for match events to your Slack or Discord World Cup sweepstakes channel
  </h5>

  <div
      class="section-expand"
      on:click={toggleHelp}
  >
    <div>Help and Information</div>
    <div class="flex-space"></div>
    <div><div class="status {helpExpanded ? 'expanded' : ''}">＋</div></div>
  </div>

  <div class="form-help collapsible-wrapper {helpExpanded ? '' : 'collapsed'}">
    <p>
      This bot posts to your Slack or Discord channel for any match event during the World Cup.
      <br>
      Events include
    </p>
    <ul>
      <li><strong>Match Starting</strong></li>
      <li><strong>Goal Scored</strong></li>
      <li><strong>Goal Disallowed</strong></li>
      <li>and <strong>Match Finishing</strong> events <i>(declaring the winner, or if the match were drawn)</i></li>
    </ul>

    <p>
      Using this service requires generating some data, and deciding on some configuration values for your use case
    </p>

    <ul>
      <li><strong>Webhook</strong> is an incoming webhook you'll need to generate for your <a href="https://api.slack.com/messaging/webhooks">Slack</a> or <a href="https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks">Discord</a> channel</li>
      <li><strong>Service</strong> is either Slack or Discord, this should be selected automatically when you enter your webhook</li>
      <li><strong>Emoji</strong> should be a comma separated list of emoji which will be randomly selected to be used in any messages for their event</li>
      <li><strong>Teams</strong> should be the name of the person allocated to each team, this can either be </li>
      <ul>
        <li>a <strong>tagged identifier</strong>, these are detected by a prefixed @. For Slack this should look something like @fbloggs and for Discord this will be their <a href="https://www.businessinsider.com/guides/tech/discord-id?r=US&IR=T">Discord user ID</a> e.g. @123456789</li>
        <li>or the person's <strong>full name</strong> such as Fred Bloggs</li>
      </ul>
    </ul>
  </div>

  <form on:submit|preventDefault>
    <div class="form-section">
      <span class="form-element">
        <label for="webhook">webhook</label>
        <input
            id="webhook"
            name="webhook"
            placeholder="https://hooks.slack.com/webhooks/..."
            on:change={hydrateService}
            bind:value={formValues.webhook}
        />
        <a
            class="button"
            on:click={fetchAndHydrateConfiguration}
            title="Fetch existing webhook configuration"
        >⬇</a>
        {#if fetchFeedback !== null}
          <small
              class="feedback {fetchFeedback.type}"
          >{fetchFeedback.message}</small>
        {/if}
      </span>
      <span class="form-element">
      <label for="service">Service</label>
      <select
          id="service"
          name="service"
          bind:value={formValues.service}
      >
        <option value="discord">Discord</option>
        <option value="slack">Slack</option>
      </select>
      </span>
    </div>

    <div class="form-section">
      <span class="form-element">
      <label for="win_emoji" title="{emojiTitle}">win emoji</label>
      <input
          id="win_emoji"
          name="win_emoji"
          placeholder="tada,confetti_ball"
          bind:value={formValues.win}
      />
      </span>

      <span class="form-element">
      <label for="score_emoji" title="{emojiTitle}">score emoji</label>
      <input
          id="score_emoji"
          name="score_emoji"
          placeholder="soccer"
          bind:value={formValues.score}
      />
      </span>

      <span class="form-element">
      <label for="draw_emoji" title="{emojiTitle}">draw emoji</label>
      <input
          id="draw_emoji"
          name="draw_emoji"
          placeholder="expressionless"
          bind:value={formValues.draw}
      />
      </span>

      <span class="form-element">
      <label for="kickoff_emoji" title="{emojiTitle}">kickoff emoji</label>
      <input
          id="kickoff_emoji"
          name="kickoff_emoji"
          placeholder="partying_face,fireworks"
          bind:value={formValues.kickOff}
      />
      </span>
    </div>

    <div
        class="section-expand"
        on:click={toggleTeams}
    >
      <div>Team Selection</div>
      <div class="flex-space"></div>
      <div><div class="status {teamsExpanded ? 'expanded' : ''}">＋</div></div>
    </div>

    <div class="collapsible-wrapper {teamsExpanded ? '' : 'collapsed'}">
      <div class="form-section">
        {#each Object.entries(teams) as [ tla, name ]}
        <span class="form-element">
        <label for="team_{tla}">{name}</label>
        <input
            id="team_{tla}"
            name="team_{tla}"
            bind:value={formValues.owners[tla]}
        />
        </span>
        {/each}
      </div>
    </div>

    <div class="submit-section">
      <button type="submit" on:click={updateConfiguration}>update</button>
      <button class="test" title="Sends a test message to verify your webhook URL" on:click={testConfiguration}>test</button>
      <button class="delete" on:click={deleteConfiguration}>delete</button>
    </div>

    {#if persistFeedback !== null}
      <small
          class="feedback {persistFeedback.type}"
      >{persistFeedback.message}</small>
    {/if}
  </form>
</div>
