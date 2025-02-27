<script lang="ts">
	import { createEventDispatcher, getContext, tick } from 'svelte'
	import type { AppInput } from '../../../inputType'
	import type { AppViewerContext } from '../../../types'
	import type RunnableComponent from '../../helpers/RunnableComponent.svelte'
	import RunnableWrapper from '../../helpers/RunnableWrapper.svelte'
	import { initOutput } from '../../../editor/appUtils'
	import { type ColumnDef, type DbType } from './utils'
	import { sendUserToast } from '$lib/toast'
	import { getDeleteInput } from './queries/delete'

	export let id: string

	const { worldStore } = getContext<AppViewerContext>('AppViewerContext')

	let outputs = initOutput($worldStore, `${id}_delete`, {
		result: undefined,
		loading: false,
		jobId: undefined
	})

	let runnableComponent: RunnableComponent
	let loading = false

	let input: AppInput | undefined = undefined

	const dispatch = createEventDispatcher()

	export async function triggerDelete(
		resource: string,
		table: string,
		columns: ColumnDef[],
		data: Record<string, any>,
		dbType: DbType
	) {
		// const datatype = tableMetaData?.find((column) => column.isprimarykey)?.datatype

		input = getDeleteInput(resource, table, columns, dbType)

		await tick()

		if (runnableComponent) {
			let ndata = {}
			columns.forEach((x) => {
				ndata[x.field] = data[x.field]
			})
			await runnableComponent?.runComponent(
				undefined,
				undefined,
				undefined,
				{ ...ndata },
				{
					done: (x) => {
						sendUserToast('Row deleted', false)
						dispatch('deleted')
					},
					cancel: () => {
						sendUserToast('Error deleting row', true)
					},
					error: () => {
						sendUserToast('Error updating row', true)
					}
				}
			)
		}
	}
</script>

<RunnableWrapper
	noInitialize
	bind:runnableComponent
	bind:loading
	componentInput={input}
	autoRefresh={false}
	render={false}
	id={`${id}_delete`}
	{outputs}
/>
