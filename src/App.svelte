<script lang="ts">
	import Player from './Player.svelte'
    import axios from 'axios'
    import CONFIG from '../config'

    let week = 0
    let results = {}
    let roster = CONFIG.roster || []
    let playerAdd = ''

    const positionsCount = { 'QB': 1, 'WR': 2, 'RB': 2, 'TE': 1, 'FL': 1 }
    const flexValues = ['WR', 'RB']

    const rankRoster = (roster, rankings) => {
        return rankings.reduce(
            (acc, player) => (
                roster.includes(player['player_name']) ?
                    [...acc, player]
                    :
                    acc
            ), []
        )
    }

    const formatPlayer = player => {
      const { player_name, pos_rank, player_team_id, player_opponent, rank_ecr, rank_ave, rank_min, rank_max, rank_std } = player
      return { player_name, pos_rank, player_team_id, player_opponent, rank_ecr, rank_ave, rank_min, rank_max, rank_std }
    }

    const sortPosition = rankedRoster => {
        return rankedRoster.reduce((output, player) => {
            const pos = player.player_position_id
            const formattedPlayer = formatPlayer(player)
            if (output[pos].length >= positionsCount[pos]) {
                if (flexValues.includes(pos)) {
                    if (output.FL.length >= positionsCount.FL) {
                        output['Next on flex'].push(formattedPlayer)
                        return output
                    }
                    output.FL.push(formattedPlayer)
                    return output
                }
                output[`Extra ${pos}'s`].push(formattedPlayer)
                return output
            }
            output[pos].push(formattedPlayer)
            return output
        }, { 'QB': [], 'WR': [], 'RB': [], 'TE': [], 'FL': [], 'Next on flex': [], "Extra QB's": [], "Extra TE's": [] }
    )}

    const getResults = () => {
        axios
            .get(
                `https://api.fantasypros.com/v2/json/nfl/2021/consensus-rankings?type=weekly&scoring=PPR&position=OP&week=${week}&experts=available`,
                {
                    headers: {
                        "x-api-key": CONFIG.fantasyProsAPIKey,
                    }
                }
            )
            .then(response => {
                const rankedRoster = rankRoster(roster, response.data.players)
                const rankedRosterByPosition = sortPosition(rankedRoster)
                results = rankedRosterByPosition
            })
            .catch(error => console.log(error))
    }
</script>

<main>
    <h2>Roster</h2>
    <p>
        {roster.join(", ")}
    </p>
    <label>
        Add player
        <input bind:value={playerAdd} />
        <button on:click={() => {
            roster = [...roster, playerAdd]
            playerAdd = ''
        }}>
            Add
        </button>
    </label>
    <label>
        Week
        <input type="number" bind:value={week} />
    </label>
    <button on:click={() => getResults()}>Get Results</button>
    <div>
        {#each Object.entries(results) as position}
            <div>
                <h2>{position[0]}</h2>
                <div class="player-row">
                    {#each position[1] as player}
                        <Player data={player}/>
                    {/each}
                </div>
            </div>
        {/each}
    </div>
</main>

<style>
	main {
		padding: 20px;
	}

    .player-row {
        display: flex;
        margin-bottom: 40px;
    }
</style>