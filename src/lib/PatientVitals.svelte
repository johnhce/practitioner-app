<script lang="ts">
  import { CLIENT_ID, REDIRECT_URL } from '../config';
  import axios from 'axios';
  import { onMount } from 'svelte';

  import { Button, ButtonGroup, Icon, Navbar, Spinner, TabContent, TabPane } from '@sveltestrap/sveltestrap';
  import { Column, Table } from '@sveltestrap/sveltestrap';
  import { Pagination, PaginationItem, PaginationLink } from '@sveltestrap/sveltestrap';

  import { add, formatRelative, subDays } from 'date-fns'
  import type { Bundle, BundleEntry, MedicationRequest, Observation, OperationOutcome } from "fhir/r4";

  export let accessToken: string
  export let baseURL: string
  export let patientId: string

  let vitalsResource: Observation | undefined = undefined

  // for datatable
  interface TableEntry {
    id: number;
    name?: string;
    date?: string;
    val?: string;
  }
  let pageSize = 10;
  let page = 1;
  let rows:Array<TableEntry> = [];

  onMount(async () => {
    const response = await axios.get(`${baseURL}/Observation?patient=${patientId}`, {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Accept': 'application/json',
      }
    });
    vitalsResource = response.data

    let entries = getObservationEntries(response.data);
    for(let i = 0; i < entries.length; i++) {
      let te = {} as TableEntry;
      te.id = i;
      te.name = entries[i].resource?.code?.text;
      let dt = entries[i]?.resource?.effectiveDateTime;
      if (dt != undefined)
        te.date = formatRelative(new Date(dt), new Date());
      else
        te.date = '';
      te.val = getObservationDisplay(entries[i].resource)?.toString();
      rows.push(te);
    }
  })

  const getObservationDisplay = (observation: Observation | undefined) => {
    if (!observation) {
      return ''
    }
    const isBp = observation.code?.coding?.find(a=>a.code === '55284-4')
    if (isBp) {
      const systolicComponent = observation.component?.find(a=>a.code?.coding?.find(b=>b.code==='8480-6'))
      const systolic = systolicComponent?.valueQuantity?.value
      const diastolicComponent = observation.component?.find(a=>a.code?.coding?.find(b=>b.code==='8462-4'))
      const diastolic = diastolicComponent?.valueQuantity?.value
      return `${systolic}/${diastolic}`
    }
    if (!observation?.valueQuantity?.unit) {
      return observation?.valueQuantity?.value
    }
    return `${observation?.valueQuantity?.value} ${observation?.valueQuantity?.unit}`
  }

  const getObservationEntries = (bundle: Bundle<Observation | OperationOutcome>): BundleEntry<Observation>[] => {
    if (!bundle?.entry) {
      return []
    }
    const results = bundle.entry?.filter((entry) => entry.resource?.resourceType === 'Observation') as BundleEntry<Observation>[];
    return results.sort((a,b)=>{
      if (a?.resource?.effectiveDateTime && b?.resource?.effectiveDateTime) {
        return new Date(b?.resource?.effectiveDateTime).getTime() - new Date(a?.resource?.effectiveDateTime).getTime();
      }
      return 0
    })
  }
</script>

<div>
  {#if vitalsResource}
  <Table rows={rows} let:row striped>
    <Column header="Name">
      {row.name}
    </Column>
    <Column header="Date">
      {row.date}
    </Column>
    <Column header="Measurement">{row.val}</Column>
  </Table>
  <Navbar>
    <!--<Pagination
    bind:pageSize
    bind:page
    totalItems={rows.length}
    pageSizeInputDisabled/> -->
    <Pagination>
      <PaginationItem>
        <PaginationLink first href="#" />
      </PaginationItem>
      <PaginationItem>
        <PaginationLink previous href="#" />
      </PaginationItem>
      <PaginationItem>
        <PaginationLink href="#">1</PaginationLink>
      </PaginationItem>
      <PaginationItem active>
        <PaginationLink href="#">2</PaginationLink>
      </PaginationItem>
      <PaginationItem>
        <PaginationLink href="#">3</PaginationLink>
      </PaginationItem>
      <PaginationItem>
        <PaginationLink next href="#" />
      </PaginationItem>
      <PaginationItem>
        <PaginationLink last href="#" />
      </PaginationItem>
    </Pagination>
  </Navbar>

  <!-- {#each getObservationEntries(labResults) as observation, i}
  <p>
    <span class="font-medium">
      {observation.resource?.code?.text}
      {#if observation?.resource?.effectiveDateTime}
        ({formatRelative(new Date(observation?.resource.effectiveDateTime), new Date())})
      {/if}
      :
    </span>
    {getObservationDisplay(observation.resource)}
  </p>
  <div class="ml-4">
  </div>
  {/each} -->
  {:else}
  <center>
    <br/><Spinner /><br/>
    Retrieving data... Please wait.
  </center>
  {/if}
</div>
