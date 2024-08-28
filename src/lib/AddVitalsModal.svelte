<script lang="ts">
  import { Button, ButtonGroup, Form, FormGroup, Icon, Input, InputGroup, InputGroupText, Modal, Label } from "@sveltestrap/sveltestrap";
  import { Container, Col, Row } from "@sveltestrap/sveltestrap";
  import VitalsCard from "./VitalsCard.svelte";
  import axios from "axios";
	import { createEventDispatcher } from 'svelte';

  export let isModalOpen: boolean = false;
  export let validated: boolean = false;
  export let accessToken: string;
  export let patientId: string;
  export let baseURL: string;

  let vitalsPulse: string = ''
  let vitalsPulseValid: boolean
  let vitalsBodyTemp: string = ''
  let vitalsBodyTempValid: boolean
  let vitalsOxySaturation: string = ''
  let vitalsOxySaturationValid: boolean
  let vitalsWeight: string = ''
  let vitalsWeightValid: boolean
  let vitalsHeightLength: string = ''
  let vitalsHeightLengthValid: boolean
  let vitalsBMI: string = ''
  let vitalsBMIValid: boolean
  let vitalsGlucose: string = ''
  let vitalsGlucoseValid: boolean
  let vitalsCreatinine: string = ''
  let vitalsCreatinineValid: boolean
  let vitalsPotassium: string = ''
  let vitalsPotassiumValid: boolean

  let dispatch = createEventDispatcher();

  const formChanged = ():boolean => {
    return !(vitalsPulseValid || vitalsBodyTempValid || vitalsOxySaturationValid ||
            vitalsWeightValid || vitalsHeightLengthValid || vitalsBMIValid || vitalsGlucoseValid ||
            vitalsCreatinineValid || vitalsPotassiumValid);
  }

  const addVitals = async () => {
    // build update data structure
    let vitalsResource = {
      "resourceType": "Observation",
      "status": "final",
      "category": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/observation-category",
              "code": "vital-signs",
              "display": "Vital Signs"
            }
          ],
          "text": "Vital Signs"
        }
      ],
      "code": {
        "coding": [
          {
            "system": "http://loinc.org",
            "code": "8331-1"
          }
        ],
        "text": "Temperature Oral"
      },
      "subject": {
        "reference": `Patient/${patientId}`
      },
      "effectiveDateTime": new Date().toISOString(),
      "valueQuantity": {
        "value": vitalsBodyTemp,
        "unit": "degC",
        "system": "http://unitsofmeasure.org",
        "code": "Cel"
      },
    }

    // call REST service
try {
    const response = await axios.post(`${baseURL}/Observation`, 
//      vitalsResource,
{
      "resourceType": "Observation",
      "status": "final",
      "category": [
        {
          "coding": [
            {
              "system": "http://terminology.hl7.org/CodeSystem/observation-category",
              "code": "vital-signs",
              "display": "Vital Signs"
            }
          ],
          "text": "Vital Signs"
        }
      ],
      "code": {
        "coding": [
          {
            "system": "http://loinc.org",
            "code": "8331-1"
          }
        ],
        "text": "Temperature Oral"
      },
      "subject": {
        "reference": `Patient/${patientId}`
      },
      "effectiveDateTime": new Date().toISOString(),
      "valueQuantity": {
        "value": vitalsBodyTemp,
        "unit": "degC",
        "system": "http://unitsofmeasure.org",
        "code": "Cel"
      },
    },
      {
        headers: {
          'Authorization': `Bearer ${accessToken}`,
          'Accept': 'application/json',
        }
      });
} catch (error:any) {
  if (error.response) {
    // The request was made and the server responded with a status code
    // that falls out of the range of 2xx
    console.log(error.response.data);
    console.log(error.response.status);
    console.log(error.response.headers);
  } else if (error.request) {
    // The request was made but no response was received
    console.log(error.request);
  } else {
    // Something happened in setting up the request that triggered an Error
    console.log('Error', error.message);
  }
  console.log(error.config);
}
    // fire an event to tell parent to refresh???
    //x BUT, parent page has a ref to another object which does the loading; fwd to that component???
    // dispatch('updateVitalsList');
    // for now, do a page reload
//x    window.location.reload()
  }
</script>

<div>
  <Modal autoFocus backdrop body={true} centered={false} fade fullscreen={false} header="Add Vitals"
         isOpen={isModalOpen} keyboard={true} returnFocusAfterClose scrollable={false} size="xl"
         unmountOnClose>
      <Form {validated} on:submit={(e) => e.preventDefault()}>
      <Container>
        <Row>
          <Col>
            <!-- <VitalsCardPulse bind:value={vitalsPulse} bind:isValid={vitalsPulseValid}/> -->
            <VitalsCard title="Pulse" icon="heart-pulse" min={10} max={200} step={1}
                unit="bpm" unitLongDesc="Beats per minute"
                bind:value={vitalsPulse} bind:isValid={vitalsPulseValid} />
          </Col>
          <Col>
            <VitalsCard title="Body Temperature" icon="cup-hot" min={30} max={60} step={0.1}
                unit="&deg;C" unitLongDesc="Temp in &deg;C"
                bind:value={vitalsBodyTemp} bind:isValid={vitalsBodyTempValid}/>
          </Col>
          <Col>
            <VitalsCard title="Oxygen Saturation" icon="0-circle" min={60} max = {100} step = {0.1}
                unit="%" unitLongDesc="Saturation percentage"
                bind:value={vitalsOxySaturation} bind:isValid={vitalsOxySaturationValid}/>
          </Col>
        </Row>
        <Row class="mt-2">
          <Col>
            <VitalsCard title="Body Weight" icon="boxes" min={1} max = {600} step = {1}
                unit="kg" unitLongDesc="Body Weight in kilograms"
                bind:value={vitalsWeight} bind:isValid={vitalsWeightValid}/>
          </Col>
          <Col>
            <VitalsCard title="Height / Length" icon="bar-chart-fill" min={10} max = {300} step = {1}
                unit="cm" unitLongDesc="Height (length) in cm"
                bind:value={vitalsHeightLength} bind:isValid={vitalsHeightLengthValid}/>
          </Col>
          <Col>
            <VitalsCard title="Body Mass Index" icon="balloon" min={12} max = {80} step = {0.1}
                unit="" unitLongDesc="Index"
                bind:value={vitalsBMI} bind:isValid={vitalsBMIValid}/>
          </Col>
        </Row>
        <Row class="mt-2">
          <Col>
            <VitalsCard title="Glucose" icon="box" min={1} max = {20} step = {0.1}
                unit="mmol/L" unitLongDesc="Glucose (mmol/L)"
                bind:value={vitalsGlucose} bind:isValid={vitalsGlucoseValid}/>
          </Col>
          <Col>
            <VitalsCard title="Creatinine" icon="c-circle" min={0} max = {5} step = {0.1}
                unit="" unitLongDesc="Creatinine level"
                bind:value={vitalsCreatinine} bind:isValid={vitalsCreatinineValid}/>
          </Col>
          <Col>
            <VitalsCard title="Potassium" icon="p-circle" min={0} max = {3} step = {0.1}
                unit="g/dL" unitLongDesc="Potassium g/dL"
                bind:value={vitalsPotassium} bind:isValid={vitalsPotassiumValid}/>
          </Col>
        </Row>
      </Container>
      <div class="mt-3" style="float:right">
        <Button color="primary" on:click="{() => {    // disabled="{formChanged}"
          validated = true;
          if ((vitalsPulseValid || vitalsPulse === '') &&
              (vitalsBodyTempValid || vitalsBodyTemp === '') &&
              (vitalsOxySaturationValid || vitalsOxySaturation === '') &&
              (vitalsWeightValid || vitalsWeight === '') &&
              (vitalsHeightLengthValid || vitalsHeightLength === '') &&
              (vitalsBMIValid || vitalsBMI === '') &&
              (vitalsGlucoseValid || vitalsGlucose === '') &&
              (vitalsCreatinineValid || vitalsCreatinine === '') &&
              (vitalsPotassiumValid || vitalsPotassium === '')) {
            addVitals();
            isModalOpen = false
          }
        }}">Add</Button>
        <Button color="danger" on:click="{() => isModalOpen = false}">Cancel</Button>
      </div>
    </Form>
  </Modal>
</div>
