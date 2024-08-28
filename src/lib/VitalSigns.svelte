<script lang="ts">
  import axios from "axios";
  import { add, formatRelative, subDays } from 'date-fns'
  import type { Bundle, BundleEntry, MedicationRequest, Observation, OperationOutcome } from "fhir/r4";

  import { Column, Spinner, Table } from "@sveltestrap/sveltestrap";
  import { DataTable, Pagination } from "carbon-components-svelte";

  export let accessToken: string
  export let baseURL: string
  export let patientId: string

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

  const getLabResults = async () => {
    const labObservationResponse = await axios.get<Bundle<Observation | OperationOutcome>>(`${baseURL}/Observation`, {
      params: {
        subject: patientId,
        category: 'vital-signs',
        _sort: '-_lastUpdated',
        // _count: 20,
        // _skip: (page-1)*20,
        // _total: 'accurate'
      },
      headers: {
        'Authorization': `Bearer ${accessToken}`,
//        'Accept': 'application/json',
//        'Prefer': 'pagination=offset-skip',
      }
    })
  let entries = getObservationEntries(labObservationResponse.data);
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

    return labObservationResponse.data
  }

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
<div class="mt-10 max-w-md mx-auto">
{#await getLabResults()}
  <center>
    <Spinner style="margin-top: 4em"/>
  </center>
{:then labResults} 
  <DataTable
    sortable
    title="Vital Signs"
    description="Your key health measurements."
    headers={[
      { key: "name", value: "Name" },
      { key: "date", value: "Date" },
      { key: "val", value: "Measurement" },
    ]}
    {pageSize}
    {page}
    {rows}
  />
  <Pagination
    bind:pageSize
    bind:page
    totalItems={rows.length}
    pageSizeInputDisabled
  />
{/await}
</div>