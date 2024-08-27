<script lang="ts">
  import { Button, ButtonGroup, Card, CardHeader, CardTitle, CardBody, Icon, Input, InputGroup } from "@sveltestrap/sveltestrap";

  export let title: string
  export let icon: string
  export let min: number
  export let max: number
  export let step: number
  export let unit, unitLongDesc: string
  export let value: string;
  export let isValid: boolean = false;
  let isInvalid: boolean = false;
</script>

<Card>
  <CardHeader>
    <ButtonGroup>
      <Icon name="{icon}"/>
      &nbsp;
      <CardTitle>{title} {#if unit}({unit}){/if}</CardTitle>
    </ButtonGroup>
  </CardHeader>
  <CardBody>
    <InputGroup>
      <Button on:click="{() => {
        value = ''
        isValid = false
        isInvalid = false
      }}"><Icon name="x-lg"/></Button>
      <Input bind:value placeholder="{unitLongDesc}" min={min} max={max} type="number" step={step}
          feedback="Must be in range {min} ... {max}"
          bind:valid={isValid} bind:invalid={isInvalid}
          on:change="{(e) => {
            const val = parseInt(e.target.value);
            if (val < min || val > max) {
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
