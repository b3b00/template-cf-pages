<script lang="ts">
	let name:string = 'world';

	import {onMount} from 'svelte';
	import {Player, Match} from 'src/types';

	onMount(async () => {
		
	});

	const getMatches = async (playerName):Promise<Match[]> => {
    	const response = await fetch(`player/${playerName}/matches`);
		console.log("response",response);
		const matches:Match[] = await response.json();
		console.log("data",matches);
		if (!matches || matches.length == 0) {
			const msg:string = `aucun matches pour le joueur ${playerName}`;
			console.log(msg);			
			return undefined;
		}
		return matches;
	}

	const getPlayer = async (playerName):Promise<Player> => {
    	const response = await fetch(`player/${playerName}`);		
		const player:Player = await response.json();	
		if (!player || !player.name || !player.ranking) {
			const msg:string = `aucun joueur ne correspond à ${playerName}`;
			console.log(msg);			
			return undefined;
		}
		return player;
	}

	const getPlayerRanking = async (playerName):Promise<number> => {
    	const response = await fetch(`player/${playerName}/ranking`);
		console.log("response",response);
		const ranking:string = await response.text();
		console.log("data",ranking);
		if (!ranking ) {
			const msg:string = `aucun classement ne correspond au joueur ${playerName}`;
			console.log(msg);			
			return undefined;
		}
		return parseFloat(ranking);
	}


	let newPlayerName:string = "";
	let newPlayerRanking:number = 500;

	let createPlayer = async () => {
		let r = await fetch("/player",
		{
			method:"POST",
			body: `name=${newPlayerName}&ranking=${newPlayerRanking}`
		});
		console.log(r);
		if (r.status != 200) {			
			console.log(`une erreur est survenue ${r.status}:${r.statusText}`);
		}
		creationVisibility = false;
		playerVisibility = true;
		playerName = newPlayerName;
		await getPlayerData();		
	}

	let player:Player = undefined;
	let playerName:string = "";	
	let playerRanking:number = 500;
	let playerMatches:Match[] = [];
	let playerVictories:number = 0;
	let playerDefeats:number = 0;

	let getPlayerData = async() => {
		player=undefined;
		playerRanking = 500;
		playerMatches = undefined;
		playerVictories = 0;
		playerDefeats = 0;
		console.log(`getting player ${playerName}`);
		player = await getPlayer(playerName);
		console.log(`player ${playerName} :: `,player);
		if (!player) {
			console.log(`no data fir player ${playerName}`);
			return;
		}
		console.log(`getting player ranking for ${playerName}`);
		playerRanking = await getPlayerRanking(playerName);
		console.log(`getting player matches for ${playerName}`);
		playerMatches = await getMatches(playerName);
		if (playerMatches && playerMatches.length > 0) {
			playerVictories = playerMatches.filter(x => x.victory).length;
			playerDefeats = playerMatches.length - playerVictories;
		}
	}

	let otherRanking = 500;
	let victory = true;
	let team = "";

	let addMatch = async () => {
		let r = await fetch(`/player/${playerName}/match`,
		{
			method:"POST",
			body: `otherRanking=${otherRanking}&victory=${(victory ? "1":"0")}&team=${team}`
		});
		console.log(r);
		if (r.status != 200) {
			console.log('zut');
			console.log(`une erreur est survenue ${r.status}:${r.statusText}`);
		}
		await getPlayerData();
	}

	let deleteMatch = async(id)=> {
		let r = await fetch(`/player/${playerName}/match/${id}`,
		{
			method:"DELETE"
		});
		console.log(r);
		if (r.status != 200) {			
			console.log(`une erreur est survenue ${r.status}:${r.statusText}`);
		}
		await getPlayerData();
	}

	let search = async (e) => {
		if (e.keyCode == 13) {
			await getPlayerData();
		}
	}

	let creationVisibility:boolean = false;
	let playerVisibility:boolean = false;

</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<h2 style="cursor: pointer;" on:click={() => creationVisibility = !creationVisibility}>Créer un joueur</h2>
<div style="display:{creationVisibility ? 'block' : 'none'}">
	<input type="text" bind:value={newPlayerName} />
	<input type="number" bind:value={newPlayerRanking}/>
	<input type="submit" on:click={createPlayer} value="créer le joueur">
</div>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<h2 style="cursor: pointer;" on:click={() => playerVisibility = ! playerVisibility}>Voir un joueur</h2> 
<div style="display:{playerVisibility ? 'block' : 'none'}">
	<input type="text" bind:value={playerName} on:keydown={search}/> 
	<input type="button" value="get" on:click={getPlayerData}/>
	{#if player && player.name}
		<h3>{player.name} : {playerRanking} ({player.ranking})</h3>
		 <ul>
			<li>{playerVictories} victoires</li>
			<li>{playerDefeats} défaites</li>
		 </ul>
		{#if  playerMatches && playerMatches.length > 0} 
		<h4>Matches</h4>
		<ul>
		{#each playerMatches as match}
			<li><i>{match.team}</i> : {match.otherRanking} {match.victory ? '👍' : '👎'} ({match.delta > 0 ?  "↗️" : "↘️"} {Math.abs(match.delta)}) <button on:click={() => deleteMatch(match.id)} value="🗑️">🗑️</button> </li>	 
		{/each}
		</ul>
		{:else}
		<span>Pas de matchs.</span>
		{/if}		
	{:else}
		<p>Aucun joueur sélectionné.</p>
	{/if}
	{#if player && player.name}
	<h4>Saisir un match :</h4>
		<div style="display:flex;flex-direction: row;flex-wrap:wrap">
			<div>
				<label for="team">Équipe :</label>	
				<input type="text" bind:value={team} id=team/>
			</div>
			<div>
				<label for="other">classement adersaire :</label>
				<input type="number" bind:value={otherRanking} id="other"/>
			</div>
			<div>
				<label for="vicotry">victoire ?</label>
				<input type="checkbox" bind:checked={victory} id="victory"/>
			</div>
			<div>
				
				<input type="button" value="✔️" on:click={addMatch}/>
			</div>	
		</div>	
	{/if}
</div>