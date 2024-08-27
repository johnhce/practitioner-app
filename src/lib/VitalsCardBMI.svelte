<script lang="ts">
  import { Button, ButtonGroup, Card, CardHeader, CardTitle, CardBody, Icon, Input, InputGroup } from "@sveltestrap/sveltestrap";

  export let value: string;
  export let isValid: boolean = false;
  let isInvalid: boolean = false;
</script>

<Card>
  <CardHeader>
    <ButtonGroup>
      <Icon name="balloon"/>
      &nbsp;
      <CardTitle>BMI</CardTitle>
    </ButtonGroup>
  </CardHeader>
  <CardBody>
    <InputGroup>
      <Button on:click="{() => {
        value = ''
        isValid = false
        isInvalid = false
      }}"><Icon name="x-lg"/></Button>
      <Input placeholder="Body mass index" min={12} max={80} type="number" step="0.1" feedback="Must be in range 12 ... 80"
      bind:valid={isValid} bind:invalid={isInvalid}
      on:change="{(e) => {
        const val = parseInt(e.target.value);
        if (val < 12 || val > 80) {
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
