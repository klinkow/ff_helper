<script>
    export let data;
</script>

<div class="player">
    <img src={data.player_square_image_url} alt="Player image."/>
    <hgroup>
        <h2>{data.player_name}</h2>
        <h3>{data.player_team_id}</h3>
    </hgroup>
    <p>Rank {data.rank_ecr}</p>
</div>

<style>
    .player {
        outline: 1px solid black;
        margin: 0 30px 30px 0;
        padding: 15px;
    }

    h2, h3 {
        margin: 0;
    }
</style>