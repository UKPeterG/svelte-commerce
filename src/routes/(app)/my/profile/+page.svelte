<style>
.frosted {
	backdrop-filter: blur(12px);
	background-color: hsla(0, 0%, 100%, 0.3);
}
</style>

<script>
import { CtrlS, SingleImageUpload } from '$lib/components'
import { page } from '$app/stores'
import { toast } from '$lib/utils'
import { UserService } from '$lib/services'
import { WhiteButton } from '$lib/ui'
import { z } from 'zod'
import Cookie from 'cookie-universal'
import dayjs from 'dayjs'
import PrimaryButton from '$lib/ui/PrimaryButton.svelte'
import SEO from '$lib/components/SEO/index.svelte'
import Skeleton from '$lib/ui/Skeleton.svelte'
import Textbox from '$lib/ui/Textbox.svelte'

export let data

const seoProps = {
	title: 'Dashboard - Edit Profile ',
	description: 'Edit the profile credentials'
}

const cookies = Cookie()

let loading = false
let formChanged = false
let err = ''
let zodError = {}

const zodProfileSchema = z.object({
	// dob: z.date({ required_error: 'First Name is required' }),
	firstName: z
		.string({ required_error: 'First Name is required' })
		.min(3, { message: 'First Name must be at least 3 characters' }),
	lastName: z
		.string({ required_error: 'Last Name is required' })
		.min(3, { message: 'Last Name must be at least 3 characters' }),
	phone: z
		.string()
		.min(10, { message: 'Phone must be at least 10 digits' })
		.max(17, { message: 'Phone must be less then 17 digits' })
})

function saveImage(detail) {
	data.profile.avatar = detail
	saveProfile()
}

function removeImage(detail) {
	data.profile.avatar = ''
	saveProfile()
}

async function saveProfile() {
	try {
		err = null
		loading = true

		let e = { ...data.profile }
		e.company = 1
		e.store = data.storeId

		if (e.dob) e.dob = dayjs(e.dob).format('YYYY-MM-DDTHH:mm')
		else e.dob = null
		// delete e.phone

		const profileData = {
			firstName: e.firstName,
			lastName: e.lastName,
			phone: e.phone
		}

		try {
			zodProfileSchema.parse(e)
			zodError = {}
		} catch (error) {
			if (error instanceof z.ZodError) {
				zodError = error.errors.reduce((acc, err) => {
					acc[err.path[0]] = err.message
					return acc
				}, {})
			}
		}

		if (Object.keys(zodError).length === 0) {
			toast('Saving Profile Info...', 'warning')

			data.profile = await UserService.updateProfileService({
				e: profileData,
				origin: $page.data.origin,
				sid: $page.data.sid,
				storeId: $page.data.storeId
			})

			if (data.profile) {
				data.profile.dob = data.profile.dob ? dayjs(data.profile.dob).format('YYYY-MM-DD') : null
				toast('Profile Info Saved.', 'success')
			}

			await cookies.set('me', data.profile, { path: '/', maxAge: 31536000 })

			// $page.data.me = data.profile
			// refreshData()
		}
	} catch (e) {
		err = e
		toast(e, 'error')
	} finally {
		loading = false
		formChanged = false
	}
}
</script>

<SEO {...seoProps} />

<section>
	<header class="mb-5 flex flex-wrap items-start justify-between gap-4">
		<h1>Profile</h1>

		<!--  Back button -->

		<div class="flex flex-wrap items-center gap-2">
			<a href="/my" class="block">
				<WhiteButton hideLoading class="group text-xs">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke-width="1.5"
						stroke="currentColor"
						class="h-5 w-5 transform transition duration-300 group-hover:-translate-x-2">
						<path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5"
						></path>
					</svg>

					<div class="flex flex-col gap-0.5 text-left">
						<span class="hidden text-xs font-normal text-zinc-500 sm:block"> Prev </span>

						<span>Dashboard</span>
					</div>
				</WhiteButton>
			</a>
		</div>
	</header>

	<div class="max-w-3xl">
		{#if loading}
			<div class="flex flex-col gap-5">
				<Skeleton />
				<Skeleton />
				<Skeleton />
			</div>
		{:else if data.profile}
			<div>
				<form class="mb-5 flex flex-col gap-4 sm:mb-10" on:submit|preventDefault="{saveProfile}">
					<div>
						<div
							class="frosted flex flex-col gap-4 rounded-lg border border-zinc-200 p-4 shadow-lg md:p-6">
							<div class="flex flex-wrap items-center gap-2">
								<SingleImageUpload
									class=""
									avatar
									folder="avatar/{(data.profile?.phone || data.profile?.email)?.replace('+', '')}"
									images="{data.profile.avatar}"
									{loading}
									on:save="{({ detail }) => saveImage(detail)}"
									on:remove="{({ detail }) => removeImage(detail)}" />

								<div class="w-full">
									{#if data.profile.email}
										<span class="mb-1 text-sm font-medium sm:text-lg lg:text-xl">
											{data.profile.email} <br />
										</span>
									{/if}

									<!-- <span class="text-xs capitalize sm:text-sm">
									Role : <b>{data.profile.role || '_'}</b>
								</span> -->
								</div>
							</div>

							<div class="flex flex-col sm:flex-row gap-2">
								<h6 class="w-52 shrink-0">First Name</h6>

								<div class="w-full">
									<Textbox
										type="text"
										placeholder="Enter First Name"
										bind:value="{data.profile.firstName}"
										on:input="{() => (formChanged = true)}" />

									{#if zodError?.firstName}
										<p class="mt-1 text-red-600">{zodError?.firstName}</p>
									{/if}
								</div>
							</div>

							<div class="flex flex-col sm:flex-row gap-2">
								<h6 class="w-52 shrink-0">Last Name</h6>

								<div class="w-full">
									<Textbox
										type="text"
										placeholder="Enter Last Name"
										bind:value="{data.profile.lastName}"
										on:input="{() => (formChanged = true)}" />

									{#if zodError?.lastName}
										<p class="mt-1 text-red-600">{zodError?.lastName}</p>
									{/if}
								</div>
							</div>

							<!-- <div class="flex flex-col sm:flex-row gap-2">
								<h6 class="w-52 shrink-0">Date Of Birth</h6>

								<div class="w-full">
									<Textbox
										type="date"
										placeholder="Enter Date Of Birth"
										bind:value="{data.profile.dob}"
										on:input="{() => (formChanged = true)}" />

									{#if zodError?.dob}
										<p class="mt-1 text-red-600">{zodError?.dob}</p>
									{/if}
								</div>
							</div> -->

							<div class="flex flex-col sm:flex-row gap-2">
								<h6 class="w-52 shrink-0">Phone</h6>

								<div class="w-full">
									<Textbox
										type="text"
										placeholder="Eg: +91000000000"
										maxlength="13"
										bind:value="{data.profile.phone}"
										on:input="{() => (formChanged = true)}" />

									{#if zodError?.phone}
										<p class="mt-1 text-red-600">{zodError?.phone}</p>
									{/if}
								</div>
							</div>
						</div>
					</div>
				</form>

				{#if data.profile.email}
					<a href="/auth/change-password?ref=/my/profile">
						<PrimaryButton>Change Password</PrimaryButton>
					</a>
				{/if}
			</div>
		{/if}
	</div>

	<CtrlS
		{loading}
		loadingMessage="Updating Profile"
		{formChanged}
		on:save="{() => saveProfile()}" />
</section>
