<script>
    import PageLayout from '../components/PageLayout.svelte'
    import SolidButton from '../components/SolidButton.svelte'
    import { warrior } from '../stores.js'
    import { validateName, validatePasswords } from '../validationUtils.js'

    export let router
    export let xfetch
    export let notifications
    export let eventTag
    export let battleId

    let warriorName = $warrior.name || ''
    let warriorEmail = ''
    let warriorPassword1 = ''
    let warriorPassword2 = ''

    $: targetPage = battleId ? `/battle/${battleId}` : '/battles'

    function createWarriorPrivate(e) {
        e.preventDefault()
        const body = {
            warriorName,
        }
        const validName = validateName(warriorName)

        let noFormErrors = true

        if (!validName.valid) {
            noFormErrors = false
            notifications.danger(validName.error, 1500)
        }

        if (noFormErrors) {
            xfetch('/api/warrior', { body })
                .then(res => res.json())
                .then(function(newWarrior) {
                    warrior.create({
                        id: newWarrior.id,
                        name: newWarrior.name,
                        rank: newWarrior.rank,
                    })

                    eventTag('register_guest', 'engagement', 'success', () => {
                        router.route(targetPage, true)
                    })
                })
                .catch(function(error) {
                    notifications.danger(
                        'Error encountered registering warrior as guest',
                    )
                    eventTag('register_guest', 'engagement', 'failure')
                })
        }
    }

    function createWarriorCorporal(e) {
        e.preventDefault()
        const body = {
            warriorName,
            warriorEmail,
            warriorPassword1,
            warriorPassword2,
        }

        const validName = validateName(warriorName)
        const validPasswords = validatePasswords(
            warriorPassword1,
            warriorPassword2,
        )

        let noFormErrors = true

        if (!validName.valid) {
            noFormErrors = false
            notifications.danger(validName.error, 1500)
        }

        if (!validPasswords.valid) {
            noFormErrors = false
            notifications.danger(validPasswords.error, 1500)
        }

        if (noFormErrors) {
            xfetch('/api/enlist', { body })
                .then(res => res.json())
                .then(function(newWarrior) {
                    warrior.create({
                        id: newWarrior.id,
                        name: newWarrior.name,
                        email: newWarrior.email,
                        rank: newWarrior.rank,
                    })

                    eventTag(
                        'register_account',
                        'engagement',
                        'success',
                        () => {
                            router.route(targetPage, true)
                        },
                    )
                })
                .catch(function(error) {
                    notifications.danger('Error encountered creating warrior')
                    eventTag('register_account', 'engagement', 'failure')
                })
        }
    }

    $: registerDisabled = warriorName === ''
    $: createDisabled =
        warriorName === '' ||
        warriorEmail === '' ||
        warriorPassword1 === '' ||
        warriorPassword2 === ''
</script>

<PageLayout>
    <div class="text-center px-2 mb-4">
        <h1 class="text-3xl md:text-4xl font-bold">Enlist to Battle</h1>
    </div>
    <div class="flex flex-wrap justify-center">
        {#if !$warrior.id}
            <div class="w-full md:w-1/2 px-4">
                <form
                    on:submit="{createWarriorPrivate}"
                    class="bg-yelloy shadow-lg rounded p-4 md:p-6 mb-4"
                    name="registerGuest">
                    <h2
                        class="font-bold text-xl md:text-2xl b-4 mb-2 md:mb-6
                        md:leading-tight text-center">
                        Register as Guest
                    </h2>

                    <div class="mb-6">
                        <label
                            class="block text-gray-700 text-sm font-bold mb-2"
                            for="yourName1">
                            Name
                        </label>
                        <input
                            bind:value="{warriorName}"
                            placeholder="Enter your name"
                            class="bg-gray-200 border-gray-200 border-2
                            appearance-none rounded w-full py-2 px-3
                            text-gray-700 leading-tight focus:outline-none
                            focus:bg-yelloy focus:border-purple-500"
                            id="yourName1"
                            name="yourName1"
                            required />
                    </div>
                    <div>
                        <div class="text-right">
                            <SolidButton
                                type="submit"
                                disabled="{registerDisabled}">
                                Register
                            </SolidButton>
                        </div>
                    </div>
                </form>
            </div>
        {/if}

        <div class="w-full md:w-1/2 px-4">
            <form
                on:submit="{createWarriorCorporal}"
                class="bg-yelloy shadow-lg rounded p-4 md:p-6 mb-4"
                name="createAccount">
                <h2
                    class="font-bold text-xl md:text-2xl mb-2 md:mb-6
                    md:leading-tight text-center">
                    Create an Account
                    <span class="text-gray-500">(optional)</span>
                </h2>

                <div class="mb-4">
                    <label
                        class="block text-gray-700 text-sm font-bold mb-2"
                        for="yourName2">
                        Name
                    </label>
                    <input
                        bind:value="{warriorName}"
                        placeholder="Enter your name"
                        class="bg-gray-200 border-gray-200 border-2
                        appearance-none rounded w-full py-2 px-3 text-gray-700
                        leading-tight focus:outline-none focus:bg-yelloy
                        focus:border-purple-500"
                        id="yourName2"
                        name="yourName2"
                        required />
                </div>

                <div class="mb-4">
                    <label
                        class="block text-gray-700 text-sm font-bold mb-2"
                        for="yourEmail">
                        Email
                    </label>
                    <input
                        bind:value="{warriorEmail}"
                        placeholder="Enter your email"
                        class="bg-gray-200 border-gray-200 border-2
                        appearance-none rounded w-full py-2 px-3 text-gray-700
                        leading-tight focus:outline-none focus:bg-yelloy
                        focus:border-purple-500"
                        id="yourEmail"
                        name="yourEmail"
                        type="email"
                        required />
                </div>

                <div class="mb-4">
                    <label
                        class="block text-gray-700 text-sm font-bold mb-2"
                        for="yourPassword1">
                        Password
                    </label>
                    <input
                        bind:value="{warriorPassword1}"
                        placeholder="Enter a password"
                        class="bg-gray-200 border-gray-200 border-2
                        appearance-none rounded w-full py-2 px-3 text-gray-700
                        leading-tight focus:outline-none focus:bg-yelloy
                        focus:border-purple-500"
                        id="yourPassword1"
                        name="yourPassword1"
                        type="password"
                        required />
                </div>

                <div class="mb-4">
                    <label
                        class="block text-gray-700 text-sm font-bold mb-2"
                        for="yourPassword2">
                        Confirm Password
                    </label>
                    <input
                        bind:value="{warriorPassword2}"
                        placeholder="Confirm your password"
                        class="bg-gray-200 border-gray-200 border-2
                        appearance-none rounded w-full py-2 px-3 text-gray-700
                        leading-tight focus:outline-none focus:bg-yelloy
                        focus:border-purple-500"
                        id="yourPassword2"
                        name="yourPassword2"
                        type="password"
                        required />
                </div>

                <div>
                    <div class="text-right">
                        <SolidButton type="submit" disabled="{createDisabled}">
                            Create
                        </SolidButton>
                    </div>
                </div>
            </form>
        </div>
    </div>
</PageLayout>
