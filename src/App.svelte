<script lang="ts">
    import Github from './lib/Github.svelte'
    import Help from './lib/Help.svelte'
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
            fetchFeedback = {type: 'success', message: 'Retrieved existing configuration'}
        }

        axios.get(`${sweepstakesApi}/configuration`, {params: {url: formValues.webhook}})
            .then(hydrateConfiguration)
            .catch((result) => {
                fetchFeedback = {type: 'error', message: result.response.data.message}
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
            persistFeedback = {type: 'error', message: 'No webhook URL detected'}

            return
        }

        if (formValues.service !== 'slack' && formValues.service !== 'discord') {
            persistFeedback = {type: 'error', message: 'No valid Service detected'}

            return
        }

        await axios.post(`${sweepstakesApi}/webhook-test`, formValues)
            .then((result) => {
                persistFeedback = {type: 'success', message: result.data.message}
            })
            .catch((result) => {
                persistFeedback = {type: 'error', message: result.response.data.message}
            })
    }

    const updateConfiguration = async function () {
        if (!formValues.webhook) {
            return
        }

        await axios.put(`${sweepstakesApi}/configuration`, formValues)
            .then((result) => {
                persistFeedback = {type: 'success', message: result.data.message}
            })
            .catch((result) => {
                persistFeedback = {type: 'error', message: result.response.data.message}
            })
    }

    const deleteConfiguration = async function () {
        if (!formValues.webhook) {
            return
        }

        await axios.delete(`${sweepstakesApi}/configuration`, {params: {url: formValues.webhook}})
            .then((result) => {
                persistFeedback = {type: 'success', message: result.data.message}
            })
            .catch((result) => {
                persistFeedback = {type: 'error', message: result.response.data.message}
            })
    }

    const formValues = {
        webhook: '',
        service: '',
        owners: {},
        win: '🎉,🎊',
        score: '⚽️,💥',
        kickOff: '🏁,🚦',
        draw: '😐,🫠',
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
    let fetchFeedback: { type: string, message: string } | null = null
    let persistFeedback: { type: string, message: string } | null = null

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

    let helpHidden: boolean = true
    const toggleHelp = () => {
        helpHidden = !helpHidden
    }

    const emojiTitle = 'Emoji should be a comma separated list of emoji randomly picked for their event'
    let teamsExpanded = true
    const toggleTeams = function () {
        teamsExpanded = !teamsExpanded
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

  <Help hidden={helpHidden} on:toggle={toggleHelp}/>

  <form on:submit|preventDefault>
    <div class="form-section">
      <span class="form-element">
        <label for="webhook" class="label-help" on:click={toggleHelp}>webhook</label>
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
      <label for="service" class="label-help" on:click={toggleHelp}>Service</label>
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
      <label for="win_emoji" class="label-help" on:click={toggleHelp}>win emoji</label>
      <input
          id="win_emoji"
          name="win_emoji"
          bind:value={formValues.win}
      />
      </span>

      <span class="form-element">
      <label for="score_emoji" class="label-help" on:click={toggleHelp}>score emoji</label>
      <input
          id="score_emoji"
          name="score_emoji"
          bind:value={formValues.score}
      />
      </span>

      <span class="form-element">
      <label for="draw_emoji" class="label-help" on:click={toggleHelp}>draw emoji</label>
      <input
          id="draw_emoji"
          name="draw_emoji"
          bind:value={formValues.draw}
      />
      </span>

      <span class="form-element">
      <label for="kickoff_emoji" class="label-help" on:click={toggleHelp}>kickoff emoji</label>
      <input
          id="kickoff_emoji"
          name="kickoff_emoji"
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
      <div>
        <div class="status {teamsExpanded ? 'expanded' : ''}">＋</div>
      </div>
    </div>

    <div class="collapsible-wrapper {teamsExpanded ? '' : 'collapsed'}">
      <div class="form-section">
        {#each Object.entries(teams) as [tla, name]}
        <span class="form-element">
        <label for="team_{tla}" class="label-help" on:click={toggleHelp}>{name}</label>
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
      <button class="test" title="Sends a test message to verify your webhook URL" on:click={testConfiguration}>test
      </button>
      <button class="delete" on:click={deleteConfiguration}>delete</button>
    </div>

    {#if persistFeedback !== null}
      <small
          class="feedback {persistFeedback.type}"
      >
        {persistFeedback.message}
      </small>
    {/if}
  </form>
</div>

<style>
    div.content {
        width: 65vw;
    }

    label.label-help {
        cursor: help;
    }
</style>
