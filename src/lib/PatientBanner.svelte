<script lang="ts">
  import { CLIENT_ID, REDIRECT_URL } from '../config';
  import axios from 'axios';
  import { onMount } from 'svelte';

  import { Button, ButtonGroup, Icon, Navbar, Spinner, TabContent, TabPane } from '@sveltestrap/sveltestrap';
  import type { Patient } from 'fhir/r4';

  export let accessToken: string
  export let baseURL: string
  export let patientId: string

  let patientResource: Patient | undefined = undefined

  onMount(async () => {
    const patientResponse = await axios.get<Patient>(`${baseURL}/Patient/${patientId}`, {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Accept': 'application/json'
      }
    });
    patientResource = patientResponse.data
  })
</script>

<div>
  <Navbar container="xl" color="primary-subtle">
    {#if patientResource}
      <h3><b>Patient:</b> {patientResource?.name?.[0]?.text}</h3>
      <h4><Icon name="calendar-event" /> <b>dob:</b> {patientResource?.birthDate}</h4>
      <h4>
        {#if patientResource?.gender === 'male'}<Icon name="person-standing" />
        {:else if patientResource?.gender === 'female'}<Icon name="person-standing-dress" />
        {:else}<Icon name="question" />
        {/if}
        <b>Gender:</b> {patientResource?.gender}</h4>
        {:else}
      <Spinner />
    {/if}
  </Navbar>
</div>
