<script>
    import VoteIcon from './icons/VoteIcon.svelte'
    import PointCard from '../components/PointCard.svelte'
    import WarriorIcon from '../components/icons/WarriorIcon.svelte'

    export let activePlanId = ''
    export let plans = []
    export let points = []
    export let warriors = []
    export let highestVote = ''

    let totalVotes = plans.find(p => p.id === activePlanId).votes.length

    // get vote average from active plan
    function getVoteAverage() {
        const activePlan = plans.find(p => p.id === activePlanId)
        let average = 0

        if (activePlan.votes.length > 0) {
            const votesToAverage = activePlan.votes
                .filter(v => v.vote !== '?')
                .map(v => {
                    const vote = v.vote === '1/2' ? 0.5 : parseInt(v.vote)
                    return vote
                })

            const sum = votesToAverage.length
                ? votesToAverage.reduce(
                      (previous, current) => (current += previous),
                  )
                : 0

            average = Math.ceil(sum / votesToAverage.length) || 0
        }

        return average
    }

    function compileVoteCounts() {
        const currentPlan = plans.find(p => p.id === activePlanId)
        return currentPlan.votes.reduce((obj, v) => {
            const currentVote = obj[v.vote] || {
                count: 0,
                voters: [],
            }
            let warriorName = 'Unknown Warrior'

            if (warriors.length) {
                const warrior = warriors.find(w => w.id === v.warriorId)
                warriorName = warrior ? warrior.name : warriorName
            }
            currentVote.voters.push(warriorName)

            currentVote.count = currentVote.count + 1

            obj[v.vote] = currentVote

            return obj
        }, {})
    }

    function calculateVotePercentage(count) {
        return Math.floor(100 * (count / totalVotes))
    }

    let average = getVoteAverage()
    $: counts = compileVoteCounts(warriors)
    let showHighestVoters = false
</script>

<div
    class="flex flex-wrap items-center text-center mb-2 md:mb-4 pt-2 pb-2
    md:pt-4 md:pb-4 bg-yelloy shadow-lg rounded text-xl">
    <div class="w-1/3 ">
        <div class="mb-2">Total Votes</div>
        {totalVotes}
        <WarriorIcon />
    </div>
    <div class="w-1/3">
        <div class="mb-2">Average</div>
        <span
            class="font-bold text-green-600 border-green-500 border p-2 rounded
            ml-2 inline-block">
            {average}
        </span>
    </div>
    <div class="w-1/3">
        <div class="mb-2">Highest</div>
        <div>
            <span
                class="font-bold text-green-600 border-green-500 border p-2
                rounded ml-2 inline-block">
                {highestVote || 0}
            </span>
            - {counts[highestVote] ? counts[highestVote].count : 0}
            <span class="relative">
                <button
                    on:mouseenter="{() => (showHighestVoters = true)}"
                    on:mouseleave="{() => (showHighestVoters = false)}"
                    class="relative leading-none"
                    title="Show Voters">
                    <WarriorIcon />
                    <span
                        class="text-sm text-right text-gray-900 font-normal w-48
                        absolute left-0 top-0 -mt-2 ml-4 bg-yelloy p-2 rounded
                        shadow-lg {showHighestVoters ? '' : 'hidden'}">
                        {#if counts[highestVote]}
                            {#each counts[highestVote].voters as voter}
                                {voter}
                                <br />
                            {/each}
                        {/if}
                    </span>
                </button>
            </span>
        </div>
    </div>
</div>

<div class="flex flex-wrap mb-4 -mx-2 mb-4 lg:mb-6">
    {#each points as point}
        <div class="w-1/4 md:w-1/6 px-2 mb-4">
            <PointCard
                results="{counts[point] || { count: 0 }}"
                isLocked="{true}"
                {point} />
        </div>
    {/each}
</div>
