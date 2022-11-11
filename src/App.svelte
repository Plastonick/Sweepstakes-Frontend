<script>
  import { onMount } from 'svelte';
  import { createForm } from 'svelte-forms-lib'
  import Github from './lib/Github.svelte'
  import axios from 'axios'

  const updateConfiguration = async function () {
    axios.get('http://192.168.1.16:8090/configuration', { params: { url: formValues.webhook }})
      .then((result) => {
        formValues.service = result.data.service
        formValues.owners = result.data.owners
        formValues.win = result.data.win
        formValues.score = result.data.score
        formValues.kickOff = result.data.kickOff
        formValues.draw = result.data.draw
      })
      .catch(() => console.error('uh oh! Something bad happened'))
  }


  const formValues = {
    webhook: '',
    service: '',
    owners: '',
    win: '',
    score: '',
    kickOff: '',
    draw: ''
  }

  const {form, handleChange, handleSubmit} = createForm({
    initialValues: {
      title: '',
      name: '',
      email: ''
    },
    onSubmit: values => {
      alert(JSON.stringify(values))
    }
  })

  const emojiTitle = 'Emoji should be a comma separated list of emoji randomly picked for their event, e.g. tada,confetti_ball,partying_face,fireworks'
  const teams = [
    {'name': 'Uruguay', 'tla': 'URU'},
    {'name': 'Germany', 'tla': 'GER'},
    {'name': 'Spain', 'tla': 'ESP'},
    {'name': 'Argentina', 'tla': 'ARG'},
    {'name': 'Ghana', 'tla': 'GHA'},
    {'name': 'Brazil', 'tla': 'BRA'},
    {'name': 'Portugal', 'tla': 'POR'},
    {'name': 'Japan', 'tla': 'JPN'},
    {'name': 'Mexico', 'tla': 'MEX'},
    {'name': 'England', 'tla': 'ENG'},
    {'name': 'United States', 'tla': 'USA'},
    {'name': 'South Korea', 'tla': 'KOR'},
    {'name': 'France', 'tla': 'FRA'},
    {'name': 'Australia', 'tla': 'AUS'},
    {'name': 'Serbia', 'tla': 'SRB'},
    {'name': 'Cameroon', 'tla': 'CMR'},
    {'name': 'Denmark', 'tla': 'DEN'},
    {'name': 'Switzerland', 'tla': 'SUI'},
    {'name': 'Ecuador', 'tla': 'ECU'},
    {'name': 'Costa Rica', 'tla': 'CRC'},
    {'name': 'Poland', 'tla': 'POL'},
    {'name': 'Croatia', 'tla': 'CRO'},
    {'name': 'Saudi Arabia', 'tla': 'KSA'},
    {'name': 'Tunisia', 'tla': 'TUN'},
    {'name': 'Senegal', 'tla': 'SEN'},
    {'name': 'Belgium', 'tla': 'BEL'},
    {'name': 'Morocco', 'tla': 'MAR'},
    {'name': 'Canada', 'tla': 'CAN'},
    {'name': 'Wales', 'tla': 'WAL'},
    {'name': 'Iran', 'tla': 'IRN'},
    {'name': 'Qatar', 'tla': 'QAT'},
    {'name': 'Netherlands', 'tla': 'NED'},
  ]
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
        on:change={handleChange}
        bind:value={formValues.score}
    />
    </span>

    <span class="form-element">
    <label for="draw_emoji" title="{emojiTitle}">draw emoji</label>
    <input
        id="draw_emoji"
        name="draw_emoji"
        placeholder="expressionless"
        on:change={handleChange}
        bind:value={formValues.draw}
    />
    </span>

    <span class="form-element">
    <label for="kickoff_emoji" title="{emojiTitle}">kickoff emoji</label>
    <input
        id="kickoff_emoji"
        name="kickoff_emoji"
        placeholder="partying_face,fireworks"
        on:change={handleChange}
        bind:value={formValues.kickOff}
    />
    </span>
  </div>

  <hr>

  <div class="wrapper">
    {#each teams as {name, tla}}
      <span class="form-element">
      <label for="team_{tla}">{name}</label>
      <input
          id="team_{tla}"
          name="team_{tla}"
          on:change={handleChange}
          bind:value={$form.tla}
      />
      </span>
    {/each}
  </div>
  <button type="submit">Submit</button>
</form>
