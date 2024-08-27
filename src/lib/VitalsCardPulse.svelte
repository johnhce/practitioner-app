<script lang="ts">
  import { Button, ButtonGroup, Card, CardHeader, CardTitle, CardBody, Icon, Input, InputGroup } from "@sveltestrap/sveltestrap";

  export let value: string;
  export let isValid: boolean = false;
  let isInvalid: boolean = false;
</script>

<Card>
  <CardHeader>
    <ButtonGroup>
      <Icon name="heart-pulse"/>
      &nbsp;
      <CardTitle>Pulse</CardTitle>
    </ButtonGroup>
  </CardHeader>
  <CardBody>
    <InputGroup>
      <Button on:click="{() => {
        value = ''
        isValid = false
        isInvalid = false
      }}"><Icon name="x-lg"/></Button>
      <Input bind:value placeholder="Beats per minute" min={10} max={200} type="number" step="1"
          feedback="Must be in range 10 ... 200"
          bind:valid={isValid} bind:invalid={isInvalid}
          on:change="{(e) => {
            const val = parseInt(e.target.value);
            if (val < 10 || val > 200) {
              isValid = false;
              if (e.target.value) isInvalid = true;
            } else if (e.target.value.length > 0) {
              isValid = true;
              isInvalid = false;
            }
          }}" />
    </InputGroup>
  </CardBody>
</Card>
