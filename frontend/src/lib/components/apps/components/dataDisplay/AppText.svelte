<script lang="ts">
	import { Clipboard } from 'lucide-svelte'
	import { copyToClipboard } from '../../../../utils'
	import Button from '../../../common/button/Button.svelte'
	import Popover from '../../../Popover.svelte'
	import type { AppInput } from '../../inputType'
	import AlignWrapper from '../helpers/AlignWrapper.svelte'
	import InputValue from '../helpers/InputValue.svelte'
	import RunnableWrapper from '../helpers/RunnableWrapper.svelte'
	import { twMerge } from 'tailwind-merge'
	import type { AppEditorContext } from '../../types'
	import { getContext } from 'svelte'

	export let id: string
	export let componentInput: AppInput | undefined
	export let horizontalAlignment: 'left' | 'center' | 'right' | undefined = 'left'
	export let verticalAlignment: 'top' | 'center' | 'bottom' | undefined = undefined
	export let configuration: Record<string, AppInput>
	export let initializing: boolean | undefined = undefined
	export let customCss: Record<'text', { class: string; style: string }> | undefined = undefined

	export const staticOutputs: string[] = ['result', 'loading']

	const { app } = getContext<AppEditorContext>('AppEditorContext')

	let result: string | undefined = undefined
	let style: 'Title' | 'Subtitle' | 'Body' | 'Caption' | 'Label' | undefined = undefined
	let copyButton: boolean

	function getComponent() {
		switch (style) {
			case 'Title':
				return 'h1'
			case 'Subtitle':
				return 'h3'
			case 'Body':
				return 'p'
			case 'Caption':
				return 'p'
			case 'Label':
				return 'label'
			default:
				return 'p'
		}
	}

	function getClasses() {
		switch (style) {
			case 'Caption':
				return 'text-sm italic text-gray-500'
			case 'Label':
				return 'font-semibold text-sm'
			default:
				return ''
		}
	}

	let component = 'p'
	let classes = ''
	$: style && (component = getComponent())
	$: style && (classes = getClasses())
</script>

<InputValue {id} input={configuration.style} bind:value={style} />
<InputValue {id} input={configuration.copyButton} bind:value={copyButton} />

<RunnableWrapper flexWrap bind:componentInput {id} bind:initializing bind:result>
	<AlignWrapper {horizontalAlignment} {verticalAlignment}>
		{#if !result || result === ''}
			<div class="text-gray-400 bg-gray-100 flex justify-center items-center h-full w-full">
				No text
			</div>
		{:else}
			<div class="flex flex-wrap gap-2 pb-0.5 overflow-x-auto">
				<svelte:element
					this={component}
					class={twMerge(
						'whitespace-pre-wrap',
						$app.css?.['textcomponent']?.['text']?.class,
						customCss?.text?.class,
						classes
					)}
					style={[$app.css?.['textcomponent']?.['text']?.style, customCss?.text?.style].join(';')}
				>
					{String(result)}
				</svelte:element>
				{#if copyButton && result}
					<Popover notClickable>
						<Button size="xs" btnClasses="!px-2" on:click={() => copyToClipboard(result)}>
							<Clipboard size={14} strokeWidth={2} />
						</Button>
						<svelte:fragment slot="text">Copy to clipboard</svelte:fragment>
					</Popover>
				{/if}
			</div>
		{/if}
	</AlignWrapper>
</RunnableWrapper>
