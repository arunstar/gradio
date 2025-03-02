<script lang="ts">
	import { createEventDispatcher } from "svelte";
	import { Upload, ModifyUpload } from "@gradio/upload";
	import type { FileData } from "@gradio/client";
	import { BlockLabel } from "@gradio/atoms";
	import { Webcam } from "@gradio/image";
	import { Video, Upload as UploadIcon } from "@gradio/icons";

	import { prettyBytes, playable } from "./utils";
	import Player from "./Player.svelte";
	import type { I18nFormatter } from "@gradio/utils";

	export let value: FileData | null = null;
	export let subtitle: FileData | null = null;
	export let sources:
		| ["webcam"]
		| ["upload"]
		| ["webcam", "upload"]
		| ["upload", "webcam"] = ["webcam", "upload"];
	export let label: string | undefined = undefined;
	export let show_label = true;
	export let mirror_webcam = false;
	export let include_audio: boolean;
	export let autoplay: boolean;
	export let root: string;
	export let i18n: I18nFormatter;
	export let active_source: "webcam" | "upload" = "webcam";
	export let handle_reset_value: () => void = () => {};

	const dispatch = createEventDispatcher<{
		change: FileData | null;
		clear?: never;
		play?: never;
		pause?: never;
		end?: never;
		drag: boolean;
		error: string;
		upload: FileData;
		start_recording?: never;
		stop_recording?: never;
	}>();

	function handle_load({ detail }: CustomEvent<FileData | null>): void {
		value = detail;
		dispatch("change", detail);
		dispatch("upload", detail!);
	}

	function handle_clear(): void {
		value = null;
		dispatch("change", null);
		dispatch("clear");
	}

	function handle_change(video: FileData): void {
		dispatch("change", video);
	}

	let dragging = false;
	$: dispatch("drag", dragging);
</script>

<BlockLabel {show_label} Icon={Video} label={label || "Video"} />
{#if value === null || value.url === undefined}
	{#if active_source === "upload"}
		<Upload
			bind:dragging
			filetype="video/x-m4v,video/*"
			on:load={handle_load}
			on:error={({ detail }) => dispatch("error", detail)}
			{root}
		>
			<slot />
		</Upload>
	{:else if active_source === "webcam"}
		<Webcam
			{mirror_webcam}
			{include_audio}
			mode="video"
			on:error
			on:capture={() => dispatch("change")}
			on:start_recording
			on:stop_recording
			{i18n}
		/>
	{/if}
{:else}
	<ModifyUpload {i18n} on:clear={handle_clear} />
	{#if playable()}
		{#key value?.url}
			<Player
				{root}
				interactive
				{autoplay}
				src={value.url}
				subtitle={subtitle?.url}
				on:play
				on:pause
				on:stop
				on:end
				mirror={mirror_webcam && active_source === "webcam"}
				{label}
				{handle_change}
				{handle_reset_value}
			/>
		{/key}
	{:else if value.size}
		<div class="file-name">{value.orig_name || value.url}</div>
		<div class="file-size">
			{prettyBytes(value.size)}
		</div>
	{/if}
{/if}

{#if sources.length > 1}
	<span class="source-selection">
		<button
			class="icon"
			aria-label="Upload video"
			on:click={() => {
				handle_clear();
				active_source = "upload";
			}}><UploadIcon /></button
		>
		<button
			class="icon"
			aria-label="Record video"
			on:click={() => {
				handle_clear();
				active_source = "webcam";
			}}><Video /></button
		>
	</span>
{/if}

<style>
	.file-name {
		padding: var(--size-6);
		font-size: var(--text-xxl);
		word-break: break-all;
	}

	.file-size {
		padding: var(--size-2);
		font-size: var(--text-xl);
	}

	.source-selection {
		display: flex;
		align-items: center;
		justify-content: center;
		border-top: 1px solid var(--border-color-primary);
		width: 95%;
		margin: 0 auto;
	}

	.icon {
		width: 22px;
		height: 22px;
		margin: var(--spacing-lg) var(--spacing-xs);
		padding: var(--spacing-xs);
		color: var(--neutral-400);
		border-radius: var(--radius-md);
	}

	.icon:hover,
	.icon:focus {
		color: var(--color-accent);
	}
</style>
