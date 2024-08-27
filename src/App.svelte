<script lang="ts">
  import { onMount } from 'svelte';
  import {CLIENT_ID, REDIRECT_URL as REDIRECT_URI,} from './config'
  import axios from 'axios';
  import pkceChallenge from "pkce-challenge";

  import PractitionerLaunch from './lib/PractitionerLaunch.svelte';
  import { Button, ButtonGroup, Icon, Navbar, Spinner, TabContent, TabPane } from '@sveltestrap/sveltestrap';
  import { colorMode, useColorMode } from '@sveltestrap/sveltestrap';
</script>

<svelte:head>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.2/font/bootstrap-icons.css">
</svelte:head>

<main>
  <PractitionerLaunch></PractitionerLaunch>
</main>
