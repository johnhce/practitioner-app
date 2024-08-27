<script lang="ts">
  import { Button, ButtonGroup, Card, CardHeader, CardTitle, CardBody, Icon, Input, InputGroup } from "@sveltestrap/sveltestrap";

  export let value: string;
  export let isValid: boolean = false;
  let isInvalid: boolean = false;
</script>

<Card>
  <CardHeader>
    <ButtonGroup>
      <Icon name="cup-hot"/>
      &nbsp;
      <CardTitle>Body Temperature</CardTitle>
    </ButtonGroup>
  </CardHeader>
  <CardBody>
    <InputGroup>
      <Button on:click="{() => {
        value = ''
        isValid = false
        isInvalid = false
      }}"><Icon name="x-lg"/></Button>
      <Input bind:value placeholder="Temp in &deg;C" min={30} max={60} type="number" step="1"
             feedback="Must be in range 30 ... 60"
             bind:valid={isValid} bind:invalid={isInvalid}
             on:change="{(e) => {
               const val = parseInt(e.target.value);
               if (val < 30 || val > 60) {
                 isValid = false;
                 if (e.target.value) isInvalid = true;
               } else {
                 isValid = true;
                 isInvalid = false;
               }
             }}" />
    </InputGroup>
  </CardBody>
</Card>
