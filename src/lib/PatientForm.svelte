<script lang="ts">
  import axios from "axios";
  import { Button, Form, FormGroup, RadioButtonGroup, RadioButton, TextInput } from "carbon-components-svelte";
  import { DatePicker, DatePickerInput } from "carbon-components-svelte";
  import Add from "carbon-icons-svelte/lib/Add.svelte";
  import Save from "carbon-icons-svelte/lib/Save.svelte";
  import Exit from "carbon-icons-svelte/lib/Exit.svelte";

  import type { Bundle, BundleEntry, Patient } from "fhir/r4";
  import { FHIR_BASE_URL } from "../config";

  import { createForm } from "felte";
  import * as yup from "yup";

  export let mode: string = 'Add'; // 'Add' or 'Edit'
  export let handleDataSubmit;
  
  // validation schema
  let formValues = {
    name: '',
    gender: '',
    dob: '',
  };

  import { createEventDispatcher } from 'svelte';
  let dispatch = createEventDispatcher();
  const sendBackData = () => {
    handleDataSubmit(formValues);
  }
const handleSendBackData = () => {
  handleDataSubmit(formValues);
}
  let maxDate: String = new Date(new Date().getDate() + 1).toISOString().split("T")[0];
  const schema = yup.object().shape({
    name: yup.string().required(),
    genderIdentity: yup.string(),
    // dob: yup.date()
    //   .nullable()
    //   .default<Date>(new Date('2000-01-01'))
    //   .required()
    //   .min(new Date('1800-01-01'), 'Date cannot be this early')
    //   .max(maxDate, 'Birth date cannot be in the future')
      // .test('format', 'Date must be in format YYYY-MM-DD',
      //   date => {
      //     if (date.toISOString().length > 0)
      //       return (date.toISOString().substring(0,10).match('/^([0-9]{4})\-([0-9]{2})\-([0-9]{2})$/') != null);
      //           })
  });

//  const errors = schema.validate(formValues);
  const { form, errors } = createForm({
      validate: async (values) => {
          try {
              await schema.validate(values, { abortEarly: false });
          } catch(err: any) {
              const errors = err.inner.reduce((res: any, value: any) => ({
                [value.path]: value.message,
                ...res,
              }), {});
              return errors;
          }
      }
  });

  const validateWithYup = () => {
    try {
      schema.validate(formValues, { abortEarly: false });
    } catch(err: any) {
      const errors = err.inner.reduce((res: any, value: any) => ({
        [value.path]: value.message,
        ...res,
      }), {});
      return errors;
    }
  }
</script>

<div>
  <h1 class="mb-10">{mode} Patient</h1>
  <TextInput id="focus-me" labelText="Name" placeholder="John Q Public" name="name" bind:value="{formValues.name}"/>
  {#if errors.name}
    <span class="text-red-500 text-sm">{errors.name}</span>
  {/if}

  <DatePicker class="mt-5" datePickerType="single" on:change dateFormat="Y-m-d">
    <DatePickerInput
      labelText="Date of birth"
      placeholder="YYYY-MM-DD"
      bind:value="{formValues.dob}"
    />
  </DatePicker>
  {#if errors.dob}
    <span class="text-red-500 text-sm">{errors.dob}</span>
  {/if}

  <FormGroup class="mt-5" legendText="Legal Gender (Sex)">
    <RadioButtonGroup name="radio-button-group" bind:selected="{formValues.gender}">
      <RadioButton value="male" labelText="Male" checked />
      <RadioButton value="female" labelText="Female" />
      <RadioButton value="other" labelText="Other" />
    </RadioButtonGroup>
  </FormGroup>
</div>
