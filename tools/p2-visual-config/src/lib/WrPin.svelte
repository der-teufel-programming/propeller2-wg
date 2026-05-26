<script lang="ts">
  import { render } from "svelte/server";
  import smartpins_raw from "./smartpins.json";

  import DropDown from "./DropDown.svelte";

  type Tuple<T, N extends number, R extends unknown[] = []> = R["length"] extends N ? R : Tuple<T, N, [T, ...R]>;

  type index = number;

  interface SmartPin {
    name: string;
    value: number;
    out_overwrite: boolean;
    dac_mode: boolean;
  }

  const smartpins: SmartPin[] = Array<SmartPin>(32);
  for (const key in smartpins_raw) {
    const smartpin: SmartPin = smartpins_raw[key];
    smartpin.value = Number("0b" + key);
    smartpins[smartpin.value] = smartpin;
  }

  interface DriveMode {
    name: string;
    value: number;
  }

  let drive_modes: DriveMode[] = [
    { value: 0b000, name: "Fast" },
    { value: 0b001, name: "1.5 kΩ" },
    { value: 0b010, name: "15 kΩ" },
    { value: 0b011, name: "150 kΩ" },
    { value: 0b100, name: "1 mA" },
    { value: 0b101, name: "100 µA" },
    { value: 0b110, name: "10 µA" },
    { value: 0b111, name: "Float" },
  ];

  interface SignalSource {
    name: string;
    value: number;
    display: (pin: number) => string;
  }

  let signal_sources: SignalSource[] = [
    {
      value: 0,
      name: "this pin's read state",
      display(pin: number): string {
        return `Pin ${pin}`;
      },
    },
    {
      value: 1,
      name: "relative +1 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin + 1)}`;
      },
    },
    {
      value: 2,
      name: "relative +2 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin + 2)}`;
      },
    },
    {
      value: 3,
      name: "relative +3 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin + 3)}`;
      },
    },
    {
      value: 4,
      name: "this pin's OUT bit from cogs",
      display(pin: number): string {
        return `OUT[${pin}]`;
      },
    },
    {
      value: 5,
      name: "relative -3 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin - 3)}`;
      },
    },
    {
      value: 6,
      name: "relative -2 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin - 2)}`;
      },
    },
    {
      value: 7,
      name: "relative -1 pin's read state",
      display(pin: number): string {
        return `Pin ${wrap_pin(pin - 1)}`;
      },
    },
  ];

  interface SignalInvert {
    name: string;
    value: boolean;
  }

  let signal_inversions: SignalInvert[] = [
    { value: false, name: "Keep" },
    { value: true, name: "Invert" },
  ];

  interface SignalFilter {
    name: string;
    value: number;
    a_display: (a: string, b: string) => string;
    b_display: (a: string, b: string) => string;
  }

  let signal_filters: SignalFilter[] = [
    {
      value: 0,
      name: "Ą, B",
      a_display(a: string, b: string): string {
        return a;
      },
      b_display(a: string, b: string): string {
        return b;
      },
    },
    {
      value: 1,
      name: "A AND B, B",
      a_display(a: string, b: string): string {
        return `${a} AND ${b}`;
      },
      b_display(a: string, b: string): string {
        return b;
      },
    },
    {
      value: 2,
      name: "A OR B, B",
      a_display(a: string, b: string): string {
        return `${a} OR ${b}`;
      },
      b_display(a: string, b: string): string {
        return b;
      },
    },
    {
      value: 3,
      name: "A XOR B, B",
      a_display(a: string, b: string): string {
        return `${a} XOR ${b}`;
      },
      b_display(a: string, b: string): string {
        return b;
      },
    },
    {
      value: 4,
      name: "A, B, both filtered using global filt0 settings",
      a_display(a: string, b: string): string {
        return `filt0(${a})`;
      },
      b_display(a: string, b: string): string {
        return `filt0(${b})`;
      },
    },
    {
      value: 5,
      name: "A, B, both filtered using global filt1 settings",
      a_display(a: string, b: string): string {
        return `filt0(${a})`;
      },
      b_display(a: string, b: string): string {
        return `filt0(${b})`;
      },
    },
    {
      value: 6,
      name: "A, B, both filtered using global filt2 settings",
      a_display(a: string, b: string): string {
        return `filt0(${a})`;
      },
      b_display(a: string, b: string): string {
        return `filt0(${b})`;
      },
    },
    {
      value: 7,
      name: "A, B, both filtered using global filt3 settings",
      a_display(a: string, b: string): string {
        return `filt0(${a})`;
      },
      b_display(a: string, b: string): string {
        return `filt0(${b})`;
      },
    },
  ];
  interface DACMode {
    name: string;
    value: number;
  }

  const dac_modes: DACMode[] = [
    { value: 0, name: "DAC 990 Ω, 3.3 V" },
    { value: 1, name: "DAC 600 Ω, 2.0 V" },
    { value: 2, name: "DAC 123.75 Ω, 3.3 V" },
    { value: 3, name: "DAC 75 Ω, 2.0 V" },
  ];

  interface ADCMode {
    name: string;
    value: number;
  }

  const adc_modes: ADCMode[] = [
    { value: 0b000, name: "GND" },
    { value: 0b001, name: "Vxxyy" },
    { value: 0b010, name: "float" },
    { value: 0b011, name: "Pin 1x" },
    { value: 0b100, name: "Pin 3.16x" },
    { value: 0b101, name: "Pin 10x" },
    { value: 0b110, name: "Pin 31.6x" },
    { value: 0b111, name: "Pin 100x" },
  ];

  interface LLClocking {
    name: string;
    value: boolean;
  }

  const clocking_modes: LLClocking[] = [
    { name: "Live", value: false },
    { name: "Clocked", value: true },
  ];

  interface LLFeedback {
    name: string;
    value: number;
  }

  const logic_feedback: LLFeedback[] = [
    { name: "None", value: 0 },
    { name: "Feedback", value: 1 },
    { name: "Adjacent-Pin", value: 2 },
  ];

  const comparator_feedback: LLFeedback[] = [
    { name: "None", value: 0 },
    { name: "Enabled", value: 1 },
  ];

  const level_comparator_feedback: LLFeedback[] = [
    { name: "None", value: 0, display: "Pin > D" },
    { name: "Local", value: 1, display: "Pin > D" },
    { name: "Separate", value: 2, display: "Adj > D" },
  ];

  type OutputEnable = "0" | "DIR";
  type DacEnable = "0" | "DIR";
  type AdcEnable = "0" | "1" | "OUT";
  type ComparatorMode = "0" | "Pin > Adj" | "Pin > D" | "Adj > D";

  interface LLMode {
    name: string;
    clockable: boolean;
    driveable: boolean;
    output_invertible: boolean;
    input_invertible: boolean;
    feedback: LLFeedback[] | null;
    dac_mode: boolean;
    adc_mode: boolean;
    dac_level: boolean;

    // results:
    get_clocked: () => boolean;
    get_low_drive: () => DriveMode | null;
    get_high_drive: () => DriveMode | null;
    get_out_inv: () => boolean | null;
    get_in_inv: () => boolean;
    get_out_enable: () => OutputEnable;
    get_dac_enable: () => DacEnable;
    get_adc_enable: () => AdcEnable;
    get_adc_mode: () => ADCMode | null;
    get_comp_mode: () => ComparatorMode;
  }

  const lowlevel_modes: LLMode[] = [
    {
      name: "Logic",
      clockable: true,
      driveable: true,
      feedback: logic_feedback,
      dac_mode: false,
      adc_mode: false,
      output_invertible: true,
      input_invertible: true,
      dac_level: false,

      get_clocked: () => ll_clocked,
      get_low_drive: () => drive_modes[ll_low_drive],
      get_high_drive: () => drive_modes[ll_high_drive],
      get_out_inv: () => ll_inv_out,
      get_in_inv: () => ll_inv_in,
      get_out_enable: () => "DIR",
      get_dac_enable: () => "0",
      get_adc_enable: () => "0",
      get_adc_mode: () => null,
      get_comp_mode: () => "0",
    },
    {
      name: "Schmitt",
      clockable: true,
      driveable: true,
      output_invertible: true,
      input_invertible: true,
      feedback: logic_feedback,
      dac_mode: false,
      adc_mode: false,
      dac_level: false,

      get_clocked: () => ll_clocked,
      get_low_drive: () => drive_modes[ll_low_drive],
      get_high_drive: () => drive_modes[ll_high_drive],
      get_out_inv: () => ll_inv_out,
      get_in_inv: () => ll_inv_in,
      get_out_enable: () => "DIR",
      get_dac_enable: () => "0",
      get_adc_enable: () => "0",
      get_adc_mode: () => null,
      get_comp_mode: () => "0",
    },
    {
      name: "Comparator",
      clockable: true,
      driveable: true,
      output_invertible: true,
      input_invertible: true,
      feedback: comparator_feedback,
      dac_mode: false,
      adc_mode: false,
      dac_level: false,

      get_clocked: () => ll_clocked,
      get_low_drive: () => drive_modes[ll_low_drive],
      get_high_drive: () => drive_modes[ll_high_drive],
      get_out_inv: () => ll_inv_out,
      get_in_inv: () => ll_inv_in,
      get_out_enable: () => "DIR",
      get_dac_enable: () => "0",
      get_adc_enable: () => "0",
      get_adc_mode: () => null,
      get_comp_mode: () => "0",
    },
    {
      name: "ADC with Optional Drive",
      clockable: false,
      driveable: true,
      output_invertible: true,
      input_invertible: false,
      feedback: null,
      dac_mode: false,
      adc_mode: true,
      dac_level: false,

      get_clocked: () => true,
      get_low_drive: () => drive_modes[ll_low_drive],
      get_high_drive: () => drive_modes[ll_high_drive],
      get_out_inv: () => ll_inv_out,
      get_in_inv: () => false,
      get_out_enable: () => "DIR",
      get_dac_enable: () => "0",
      get_adc_enable: () => "1",
      get_adc_mode: () => adc_modes[ll_adc_mode],
      get_comp_mode: () => "0",
    },
    {
      name: "DAC with Optional DAC",
      clockable: false,
      driveable: false,
      output_invertible: false,
      input_invertible: false,
      feedback: null,
      dac_mode: true,
      adc_mode: false,
      dac_level: true,

      get_clocked: () => true,
      get_low_drive: () => null,
      get_high_drive: () => null,
      get_out_inv: () => null,
      get_in_inv: () => false,
      get_out_enable: () => "0",
      get_dac_enable: () => "DIR",
      get_adc_enable: () => "OUT",
      get_adc_mode: () => adc_modes[0b011],
      get_comp_mode: () => "0",
    },
    {
      name: "Level Comparator",
      clockable: true,
      driveable: false,
      output_invertible: false,
      input_invertible: false,
      feedback: level_comparator_feedback,
      dac_mode: false,
      adc_mode: false,
      dac_level: true,

      get_clocked: () => ll_clocked,
      get_low_drive: () => drive_modes[0b001],
      get_high_drive: () => drive_modes[0b001],
      get_out_inv: () => null, // TODO!
      get_in_inv: () => false,
      get_out_enable: () => "DIR",
      get_dac_enable: () => "0",
      get_adc_enable: () => "0",
      get_adc_mode: () => null,
      get_comp_mode: () => level_comparator_feedback[ll_feedback].display,
    },
  ];

  let pin_index: number = $state(0);
  let a_inverted: boolean = $state(false);
  let a_source: number = $state(0);

  let b_inverted: boolean = $state(false);
  let b_source: number = $state(0);

  let sig_filter: number = $state(0);

  let ll_mode: LLMode = $state(lowlevel_modes[0]);

  let ll_clocked: boolean = $state(false);
  let ll_low_drive: index = $state(0);
  let ll_high_drive: index = $state(0);
  let ll_inv_in: boolean = $state(false);
  let ll_inv_out: boolean = $state(false);
  let ll_feedback: index = $state(0);
  let ll_dac_mode: index = $state(0);
  let ll_adc_mode: index = $state(0);
  let ll_dac_level: number = $state(0);

  let smart_mode: number = $state(0);

  let count: number = $state(0);
  const increment = () => {
    count += 1;
  };

  function render_as_bin(value: boolean | number, bits: number): string {
    if (value === true) {
      return "0".repeat(bits - 1) + "1";
    }
    if (value === false) {
      return "0".repeat(bits);
    }

    return value.toString(2).padStart(bits, "0").substring(0, bits);
  }

  function wrap_pin(pos: number): number {
    while (pos < 0) {
      pos += 64;
    }
    pos = pos % 64;
    return pos;
  }

  function input_display(source: number, inverted: boolean): string {
    let out = signal_sources[source].display(pin_index);
    if (inverted) {
      out += ", inverted";
    }
    return out;
  }
</script>

<h3>Configuration</h3>

<table>
  <thead>
    <tr>
      <td></td>
      <td>Configuration</td>
      <td>Representation</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Target Pin</th>
      <td>
        <input type="number" min="0" max="63" bind:value={pin_index} />
      </td>
      <td> </td>
    </tr>

    <tr>
      <th>A Input</th>
      <td>
        <DropDown bind:value={a_source} items={signal_sources} />
        <DropDown bind:value={a_inverted} items={signal_inversions} />
      </td>
      <td><code>{render_as_bin(a_inverted, 1)}{render_as_bin(a_source, 3)}</code></td>
    </tr>

    <tr>
      <th>B Input</th>
      <td>
        <DropDown bind:value={b_source} items={signal_sources} />
        <DropDown bind:value={b_inverted} items={signal_inversions} />
      </td>
      <td><code>{render_as_bin(b_inverted, 1)}{render_as_bin(b_source, 3)}</code></td>
    </tr>

    <tr>
      <th>Filtering</th>
      <td>
        <DropDown bind:value={sig_filter} items={signal_filters} />
      </td>
      <td><code>{render_as_bin(sig_filter, 3)}</code></td>
    </tr>

    <tr>
      <th>Low Level Mode</th>
      <td>
        <select bind:value={ll_mode}>
          {#each lowlevel_modes as mode}
            <option value={mode}>{mode.name}</option>
          {/each}
        </select>
      </td>
    </tr>

    {#if ll_mode.driveable}
      <tr class="detail">
        <th>Low Drive</th>
        <td>
          <DropDown items={drive_modes} bind:value={ll_low_drive} />
        </td>
      </tr>

      <tr class="detail">
        <th>High Drive</th>
        <td>
          <DropDown items={drive_modes} bind:value={ll_high_drive} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.input_invertible}
      <tr class="detail">
        <th>Invert Input</th>
        <td>
          <DropDown items={signal_inversions} bind:value={ll_inv_in} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.output_invertible}
      <tr class="detail">
        <th>Invert Output</th>
        <td>
          <DropDown items={signal_inversions} bind:value={ll_inv_out} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.clockable}
      <tr class="detail">
        <th>Clocked</th>
        <td>
          <DropDown items={clocking_modes} bind:value={ll_clocked} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.feedback !== null}
      <tr class="detail">
        <th>Feedback</th>
        <td>
          <DropDown bind:value={ll_feedback} items={ll_mode.feedback} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.dac_mode}
      <tr class="detail">
        <th>DAC Mode</th>
        <td>
          <DropDown bind:value={ll_dac_mode} items={dac_modes} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.adc_mode}
      <tr class="detail">
        <th>ADC Mode</th>
        <td>
          <DropDown bind:value={ll_adc_mode} items={adc_modes} />
        </td>
      </tr>
    {/if}

    {#if ll_mode.dac_level}
      <tr class="detail">
        <th>DAC Level</th>
        <td>
          <input type="number" min="0" max="255" bind:value={ll_dac_level} />
        </td>
      </tr>
    {/if}

    <tr>
      <th>Smart Mode</th>
      <td>
        <DropDown list={true} items={smartpins} bind:value={smart_mode} />
      </td>
      <td><code>{render_as_bin(smart_mode, 5)}</code></td>
    </tr>
  </tbody>
</table>

<h3>Result</h3>

<table>
  <tbody>
    <tr>
      <th>Config Mask</th>
      <td>
        <code>
          %{render_as_bin(a_inverted, 1)}{render_as_bin(a_source, 3)}_{render_as_bin(b_inverted, 1)}{render_as_bin(
            b_source,
            3,
          )}_{render_as_bin(sig_filter, 3)}_MMMMMMMMMMMMM_TT_{render_as_bin(smart_mode, 5)}_0
        </code>
      </td>
    </tr>

    <tr>
      <th>CIOHHHLLL</th>
      <td>
        <code>%CIOHHHLLL</code>
      </td>
    </tr>

    <tr class="detail">
      <th>Clocked</th>
      <td>{ll_mode.get_clocked()}</td>
    </tr>

    <tr class="detail">
      <th><code>IN</code></th>
      <td>{ll_mode.get_in_inv()}</td>
    </tr>

    <tr class="detail">
      <th><code>OUT</code></th>
      <td>{ll_mode.get_out_inv()}</td>
    </tr>

    <tr class="detail">
      <th><code>H</code> Drive</th>
      <td>{ll_mode.get_high_drive().name}</td>
    </tr>

    <tr class="detail">
      <th><code>L</code> Drive</th>
      <td>{ll_mode.get_low_drive().name}</td>
    </tr>

    <tr>
      <th>Output Enable</th>
      <td>{ll_mode.get_out_enable()}</td>
    </tr>

    <tr>
      <th>DAC Enable</th>
      <td>{ll_mode.get_dac_enable()}</td>
    </tr>

    <tr>
      <th>ADC Enable</th>
      <td>{ll_mode.get_adc_enable()}</td>
    </tr>

    <tr>
      <th>ADC Mode</th>
      <td>{ll_mode.get_adc_mode()}</td>
    </tr>

    <tr>
      <th>Comparator</th>
      <td>{ll_mode.get_comp_mode()}</td>
    </tr>

    <tr>
      <th>Pin Properties</th>
      <td>
        {#if pin_index % 2 == 0}even{:else}odd{/if}
      </td>
    </tr>

    <tr>
      <th>A Input</th>
      <td
        >{signal_filters[sig_filter].a_display(
          input_display(a_source, a_inverted),
          input_display(b_source, b_inverted),
        )}</td
      >
    </tr>

    <tr>
      <th>B Input</th>
      <td
        >{signal_filters[sig_filter].b_display(
          input_display(a_source, a_inverted),
          input_display(b_source, b_inverted),
        )}</td
      >
    </tr>
  </tbody>
</table>

<style lang="scss">
  table {
    border-collapse: collapse;
    width: 100%;
  }

  table tr {
    border: 1px solid gray;
  }

  table tr th,
  table tr td {
    padding: 0.25rem 0.5rem;
  }

  table tr th {
    white-space: nowrap;
  }

  table tbody tr th {
    text-align: left;
  }

  table tbody tr.detail th {
    padding-left: 2rem;
  }
</style>
