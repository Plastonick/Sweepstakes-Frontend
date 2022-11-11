<script>
  import { onMount } from 'svelte';
  import Github from './lib/Github.svelte'
  import axios from 'axios'

  const updateConfiguration = async function () {
    const hydrateConfiguration = (result) => {
      formValues.service = result.data.service
      formValues.owners = result.data.owners
      formValues.win = result.data.win
      formValues.score = result.data.score
      formValues.kickOff = result.data.kickOff
      formValues.draw = result.data.draw
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

    axios.get('http://127.0.0.1:8090/configuration', { params: { url: formValues.webhook }})
      .then(hydrateConfiguration)
      .catch(hydrateService)
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
    URU: 'Uruguay',
    GER: 'Germany',
    ESP: 'Spain',
    ARG: 'Argentina',
    GHA: 'Ghana',
    BRA: 'Brazil',
    POR: 'Portugal',
    JPN: 'Japan',
    MEX: 'Mexico',
    ENG: 'England',
    USA: 'United States',
    KOR: 'South Korea',
    FRA: 'France',
    AUS: 'Australia',
    SRB: 'Serbia',
    CMR: 'Cameroon',
    DEN: 'Denmark',
    SUI: 'Switzerland',
    ECU: 'Ecuador',
    CRC: 'Costa Rica',
    POL: 'Poland',
    CRO: 'Croatia',
    KSA: 'Saudi Arabia',
    TUN: 'Tunisia',
    SEN: 'Senegal',
    BEL: 'Belgium',
    MAR: 'Morocco',
    CAN: 'Canada',
    WAL: 'Wales',
    IRN: 'Iran',
    QAT: 'Qatar',
    NED: 'Netherlands'
  }

  const handleSubmit = () => {}

  const emojiTitle = 'Emoji should be a comma separated list of emoji randomly picked for their event, e.g. tada,confetti_ball,partying_face,fireworks'
</script>

<Github/>

<form on:submit={handleSubmit}>
  <div class="wrapper">
    <span class="form-element">
      <label for="webhook">webhook</label>
      <input
          id="webhook"
          name="webhook"
          placeholder="https://hooks.slack.com/services/..."
          on:change={updateConfiguration}
          bind:value={formValues.webhook}
      />
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

  <hr>

  <div class="wrapper">
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
  <button type="submit">Submit</button>
</form>
