<script lang="ts">
  import { CLIENT_ID, REDIRECT_URL } from '../config';
  import axios from 'axios';
  import { onMount } from 'svelte';
  import pkceChallenge from "pkce-challenge";

  import { Button, ButtonGroup, Icon, Modal, Nav, Navbar, Spinner, TabContent, TabPane } from '@sveltestrap/sveltestrap';
  import { FormGroup, Input } from '@sveltestrap/sveltestrap';
  import PatientBanner from './PatientBanner.svelte';
  import PatientDetails from './PatientDetails.svelte';
  import PatientVitals from './PatientVitals.svelte';
  import AddVitalsModal from './AddVitalsModal.svelte';
  import VitalSigns from './VitalSigns.svelte';

  let iss: string
  let launch: string
  let authEndpoint: string
  let redirectUrl: string
  let token: any
  let isAddModalOpen: boolean = false;
  let validated = false;

  onMount(async () => {
    const launchUrl = new URL(window.location.href)
    iss = launchUrl.searchParams.get('iss') || ''
    launch = launchUrl.searchParams.get('launch') || ''

    const tokenJSON = localStorage.getItem('token')
    if (tokenJSON) {
      token = JSON.parse(tokenJSON)
    }

    if (token) {
      return
    }

    const code = launchUrl.searchParams.get('code')
    if (code) {
      // we're coming back through after logging in
      // get token
      const tokenResponseData = await makeTokenRequest({code: code, tokenUrl: localStorage.getItem("token_endpoint") || ''})
      token = tokenResponseData
      localStorage.removeItem('token_endpoint')
      localStorage.setItem('token', JSON.stringify(token))
      return
    } else {
      const smartConfigResponse = await axios.get(iss+'/.well-known/smart-configuration')
      const smartConfig = smartConfigResponse.data
      authEndpoint = smartConfig.authorization_endpoint as string
      const tokenEndpoint = smartConfig.token_endpoint as string
      localStorage.setItem("token_endpoint", tokenEndpoint);
      //x for prod, keep a cache: map of issuer,smartConfig

      redirectUrl = await constructAuthUrl({authUrl: authEndpoint,
        scopes: 'openid fhirUser launch user/Observation.read user/Onservation.write user/Patient.read user/Patient.write'});
      window.location.href = redirectUrl;
    }
  })

  const makeTokenRequest = async ({
        code,
        tokenUrl,
    }: {
        code: string;
        tokenUrl: string;
    }) => {
        const code_verifier = localStorage.getItem("code_verifier");
        const data = new URLSearchParams();
        data.append("grant_type", "authorization_code");
        data.append("code", code);
        data.append("client_id", CLIENT_ID);
        data.append("redirect_uri", REDIRECT_URL);
        data.append("code_verifier", code_verifier as string);
        const tokenRes = await axios.post(tokenUrl, data, {
            headers: {
                "Content-Type": "application/x-www-form-urlencoded",
            },
        });
        return tokenRes.data;
    };

    const constructAuthUrl = async ({ authUrl, scopes }: { authUrl: string, scopes: string }) => {
      const { code_verifier, code_challenge } = await pkceChallenge();
      console.log(code_verifier, code_challenge);

      localStorage.setItem("code_verifier", code_verifier);

      const url = new URL(authUrl);

      url.searchParams.append("response_type", "code");
      url.searchParams.append("client_id", CLIENT_ID);
      url.searchParams.append("redirect_uri", REDIRECT_URL);
      url.searchParams.append("scope", scopes);
      url.searchParams.append("launch", launch);
      url.searchParams.append("aud", iss);
      url.searchParams.append("code_challenge", code_challenge);
      url.searchParams.append("code_challenge_method", "S256");

      return url.href;
	  };
  </script>

<div>
  {#if token}
    {#if token.need_patient_banner}
      <PatientBanner accessToken={token.access_token}
                     patientId={token.patient}
                     baseURL={iss}/><!--'https://fhir-open.cerner.com/r4/ec2458f2-1e24-41c8-b71b-0e701af7583d'-->
                   <!-- was iss for baseURL -->
    {/if}
    <Navbar container="xl" color="primary-subtle" light={true}>
      <h5>Patient Vitals</h5>
      <Nav class="ms-auto" navbar>
        <Button on:click="{() => isAddModalOpen = true}"><Icon name="heart-pulse" /> Add Vitals...</Button>
      </Nav>
    </Navbar>
    <PatientVitals accessToken={token.access_token}
                   patientId={token.patient}
                   baseURL={iss}/>
                   <!-- was iss for baseURL -->
    <AddVitalsModal bind:isModalOpen={isAddModalOpen}
                   accessToken={token.access_token}
                   patientId={token.patient}
                   baseURL={iss}/>
  {:else}
    <center>
      <Spinner style="margin-top: 4em"/>
    </center>
  {/if}
</div>
