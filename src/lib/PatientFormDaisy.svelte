<script lang="ts">
  import axios from "axios";
//  import { DateInput, DatePicker, localeFromDateFnsLocale } from 'date-picker-svelte';

  import type { Bundle, BundleEntry, Patient } from "fhir/r4";
  import { FHIR_BASE_URL } from "../config";

  import { createForm } from "felte";
  import * as yup from "yup";

  import { createEventDispatcher } from 'svelte';

  export let mode: string = 'Add'; // 'Add' or 'Edit'
  export let patientId: string;

	let dispatch = createEventDispatcher();

  let maxDate: String = new Date(new Date().getDate() + 1).toISOString().split("T")[0];
  const schema = yup.object().shape({
    name: yup.string().required(),
    dob: yup.date()
      .nullable()
      .default<Date>(new Date('2000-01-01'))
      .required()
      .min(new Date('1800-01-01'), 'Date cannot be this early')
      .max(maxDate, 'Birth date cannot be in the future')
      .test('format', 'Date must be in format YYYY-MM-DD',
        date => {
          if (date.toISOString().length > 0)
            return (date.toISOString().substring(0,10).match('/^([0-9]{4})\-([0-9]{2})\-([0-9]{2})$/') != null);
                })
  });

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

  const addPatient = async () => {
    const patientResponse = await axios.post<Patient>(`${FHIR_BASE_URL}/Patient/${patientId}`, {
      //x body goes here
    });
alert(JSON.stringify(patientResponse.data));
		dispatch('returnHome', 1);
    return patientResponse.data
  }

  const updatePatient = async () => {
    const patientResponse = await axios.put<Patient>(`${FHIR_BASE_URL}/Patient/${patientId}`, {
      //x body goes here
    });
alert(JSON.stringify(patientResponse.data));
    dispatch('returnHome', 1);
    return patientResponse.data
  }

  const returnToHome = () => {
    window.localStorage.href = '/patient';
  }
</script>

<div class="flex flex-col items-center justify-center w-screen h-screen bg-gray-200 text-gray-700">
  <h1 class="mb-5">{mode} Patient</h1>
  <form use:form action="#" method="post" on:submit="{(ev) => ev.preventDefault()}">
    <label class="input input-bordered flex items-center gap-2">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 16 16"
        fill="currentColor"
        class="h-4 w-4 opacity-70">
        <path
          d="M8 8a3 3 0 1 0 0-6 3 3 0 0 0 0 6ZM12.735 14c.618 0 1.093-.561.872-1.139a6.002 6.002 0 0 0-11.215 0c-.22.578.254 1.139.872 1.139h9.47Z" />
      </svg>
      <input type="text" class="grow" placeholder="Name" name="name"/>
    </label>
    {#if errors.name}
      <span class="text-red-500 text-sm">{errors.name}</span>
    {/if}

    <label class="input input-bordered flex items-center gap-2 mt-3">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
        <path stroke-linecap="round" stroke-linejoin="round" d="M6.75 3v2.25M17.25 3v2.25M3 18.75V7.5a2.25 2.25 0 0 1 2.25-2.25h13.5A2.25 2.25 0 0 1 21 7.5v11.25m-18 0A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75m-18 0v-7.5A2.25 2.25 0 0 1 5.25 9h13.5A2.25 2.25 0 0 1 21 11.25v7.5m-9-6h.008v.008H12v-.008ZM12 15h.008v.008H12V15Zm0 2.25h.008v.008H12v-.008ZM9.75 15h.008v.008H9.75V15Zm0 2.25h.008v.008H9.75v-.008ZM7.5 15h.008v.008H7.5V15Zm0 2.25h.008v.008H7.5v-.008Zm6.75-4.5h.008v.008h-.008v-.008Zm0 2.25h.008v.008h-.008V15Zm0 2.25h.008v.008h-.008v-.008Zm2.25-4.5h.008v.008H16.5v-.008Zm0 2.25h.008v.008H16.5V15Z" />
      </svg>
      <input type="text" class="grow" name="dob" placeholder="YYYY-MM-DD" /><!-- format="yyyy-MM-dd"/-->
    </label>
    {#if errors.dob}
      <span class="text-red-500 text-sm">{errors.dob}</span>
    {/if}

    <p style="margin-top: 20px"/>
    <label>Legal Gender (sex)
      <div class="form-control">
        <label class="label cursor-pointer">
          <span class="label-text">Male</span>
          <input type="radio" name="gender" class="radio" checked value="male"/>
        </label>
        <label class="label cursor-pointer">
          <span class="label-text">Female</span>
          <input type="radio" name="gender" class="radio" value="female"/>
        </label>
        <label class="label cursor-pointer">
          <span class="label-text">Other</span>
          <input type="radio" name="gender" class="radio" value="other"/>
        </label>
      </div>
    </label>

    <!-- <DatePicker datePickerType="single" on:change dateFormat="">
      <label for="dob">
        Date of birth
        <DateInput name="dob" format="yyyy-MM-dd"/>
          placeholder="YYYY-MM-DD"
        />
      </label>
    </DatePicker>
    {#if errors.dob}
      <span class="text-red-500 text-sm">{errors.dob}</span>
    {/if} -->

    <p style="margin-top: 20px"/>

    {#if mode === 'Add'}
      <button class="btn btn-primary" on:click={addPatient}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
        </svg>
        Add
      </button>
    {:else}
      <button class="btn btn-primary" on:click={updatePatient}>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 9V5.25A2.25 2.25 0 0 1 10.5 3h6a2.25 2.25 0 0 1 2.25 2.25v13.5A2.25 2.25 0 0 1 16.5 21h-6a2.25 2.25 0 0 1-2.25-2.25V15M12 9l3 3m0 0-3 3m3-3H2.25" />
        </svg>
        Update
      </button>
    {/if}
    <button class="btn btn-secondary" on:click={returnToHome}>
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
        <path stroke-linecap="round" stroke-linejoin="round" d="M6 18 18 6M6 6l12 12" />
      </svg>
      Cancel
    </button>
  </form>
</div>
