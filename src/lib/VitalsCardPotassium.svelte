<script lang="ts">
  import { Button, ButtonGroup, Card, CardHeader, CardTitle, CardBody, Icon, Input, InputGroup } from "@sveltestrap/sveltestrap";

  export let value: string;
  export let isValid: boolean = false;
  let isInvalid: boolean = false;
</script>

<Card>
  <CardHeader>
    <ButtonGroup>
      <Icon name="p-circle"/>
      &nbsp;
      <CardTitle>Potassium</CardTitle>
    </ButtonGroup>
  </CardHeader>
  <CardBody>
    <InputGroup>
      <Button on:click="{() => {
        value = ''
        isValid = false
        isInvalid = false
      }}"><Icon name="x-lg"/></Button>
      <Input placeholder="Potassium g/dL" min={0} max={3} type="number" step="0.1" feedback="Must be in range 0 ... 3"
      bind:valid={isValid} bind:invalid={isInvalid}
      on:change="{(e) => {
        const val = parseInt(e.target.value);
        if (val < 1 || val > 3) {
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
