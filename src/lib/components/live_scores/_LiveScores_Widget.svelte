<!-- ===============
	COMPONENT JS (w/ TS)
==================== -->
<script lang="ts">
	// ... svelte-imports;
	import { onMount } from 'svelte';
	import { fade } from 'svelte/transition';
	import { browser, dev } from '$app/env';

	// ... external modules imports;
	import ColorThief from 'colorthief/dist/color-thief.mjs';

	// ... external components import;
	// ... external `exports` imports;
	import { db_real } from '$lib/firebase/init';
	import { ref, onValue } from 'firebase/database';
	import { getUserLocation } from "$lib/geoJs/init"

	// ... DECLARING TYPESCRIPT-TYPES imports;
	import type { fixture } from '$lib/store/vote_fixture';
	import type { FixtureResponse } from '$lib/model/interface-fixture';
	import type { SelectedFixutre, SelectedFixture_VoteUpdate_Response, TranslationsResponse, LiveScoreGame, LiveScoreLeagueGame, LiveScoreLeague, DayName } from '$lib/model/response_models';
	import { page } from '$app/stores'
	import { post } from '$lib/api/utils'
	import type { GeoJsResponse } from "$lib/model/geo-js-interface"
import { xlink_attr } from 'svelte/internal';

	// ... key component assets;
	import blinker from './assets/blinker.gif'
	import play from './assets/play.svg'
	let TABLE_GAMES: LiveScoreLeague[] = [];
	let currentDay = 0;
	let currentSelection = 0;
	let currentTable = 'livescores_today';
	let lastTable = 'livescores_today';
	let totalGames = 0;

	
	// ... widget-language-declaration;
	let server_side_language: string = 'en';
	// ... language-translation-declaration;
	$: if ($page.params.lang === undefined) {
		server_side_language = 'en';
	} else {
		server_side_language = $page.params.lang;
	}

	// ... Listen To Real-Time Firebase ODDS Updates [WORKING]
	async function listenRealTimeOddsChange(table:string): Promise < LiveScoreLeague[] > {
		console.log("oi");
		totalGames = 0;
		let data: LiveScoreLeague[] = [];
		onValue(ref(db_real,"livescores_table/"+table), (snapshot) => {
			// ... DEBUGGING; 
			data =[];
			let games: LiveScoreGame[] = snapshot.val();	
			console.log(table); 
	
			for(var g in games){  
				const newGame:  LiveScoreLeagueGame={
					id : games[g].id,
					visitorteam: games[g].visitorteam,
					localteam : games[g].localteam,
					localScore : games[g].localteam_score,
					visitorScore : games[g].visitorteam_score,
					minute: games[g].minute,
					medias: games[g].medias,
					status: games[g].status,
					links: games[g].links,
					tips: games[g].tips,
					hour: games[g].starting_at.split(' ')[1].slice(0,-3)
				}; 

				var l = data.filter(x=>x.id === games[g].league_id)[0];
				if(l==null){
					const league : LiveScoreLeague={
						games: [],
						id : games[g].league_id,
						flag : games[g].flag,
						name:games[g].league
					};
					data.push(league); 
					l = league;
				} 

				totalGames++;
				l.games.push(newGame);
			}

			TABLE_GAMES = data;
		});
		return data;
	}

	var dayNames = ['dom','seg','ter','qua','qui','sex','sab'];
	var tableNames = ['livescores_3daysago','livescores_2daysago','livescores_yesterday','livescores_today','livescores_tomorrow','livescores_2days','livescores_3days'];
	let LIST_DAYS: DayName[] = [];
    for(let i = -3;i < 4;i++){ 
		let day: Date = new Date();
        day.setDate(day.getDate() + i);
		LIST_DAYS.push({name:dayNames[day.getDay()],day:day.getDate(),sel:i,table:tableNames[day.getDay()]});
    } 


</script>
 
<!-- ===============
  COMPONENT HTML 
==================== -->
<p class="s-20 m-b-10 color-white">
	Live Scores
</p>
<div class="game-table-container">
	<div class="base">
		<div class="dayList">
				{#each LIST_DAYS as d}
					<div class="day {d.sel == currentDay ? 'current sel':''}" on:click="{()=>{currentDay = d.sel;currentTable=d.table}}">
						<div>{d.name}</div>
						<div class="dayNumber">{d.day}</div>
					</div>
				{/each}
		</div>
		</div>
	{#await listenRealTimeOddsChange(currentTable)}
	{:then data} 
	<div class="betarena-menu" style=""><span class="lnkAll {currentSelection == 0?'lnkSelected':''}" on:click="{()=>{currentSelection =0;currentTable = lastTable}}">All ({totalGames})</span>
		<span class="lnkLive {currentSelection == 1?'lnkSelected':''}" on:click="{()=>{currentSelection =1;lastTable=currentTable;currentTable='livescores_now'}}">Live (<span class="numLive">1</span>)</span></div>
		<div class="leaguesList">
		{#each TABLE_GAMES as d}
		<div class="league"> 
			<div class="league-title">
			<div class="icon-league">
				<img class="img-flag" src="{d.flag}" width="19" height="12" alt="League Country Flag">
				<span class="league-name">{d.name}</span>
			</div>
		</div>
		<div class="game-list">

			{#each d.games as g}
			<div class="league-body {g.status=='LIVE' ? 'in-progress':''} {(g.status == "NS"  || g.status == "POSTP" || g.status == "TBA" || g.status == "CANCL") ? 'not-started':''}">
				<div class="game-col-1" title="">
					{#if g.status != "LIVE" && g.status != "HT" }
						<div class="game-time" style="">{g.hour}</div>
					{:else}
						<div class="game-time" style="">{g.minute} <img style="position: absolute;" src="{blinker}" alt="Blinker"></div>
					{/if}
					<div class="game-status" style="display:none;"></div>
				</div>
				<a class="game-teams" href="https://www.betarena.com/brentford-vs-manchester-united-livescore-preview-2/">
					<div class="team first loser"><span class="teamname">{g.localteam}</span><span class="cards"></span></div>
					<div class="team second "><span class="teamname">{g.visitorteam}</span><span class="cards"></span></div></a>
				{#if g.medias > 0}
				<a class="game-media" href="https://www.betarena.com/brentford-vs-manchester-united-livescore-preview-2/" >
					<img alt="Game Media" src="{play}" width="28" height="28"> 
				</a>    
				{/if}  

				{#if g.tips != null && g.tips[server_side_language] != undefined}
					<a class="table-game-tip" style="" href="{g.tips[server_side_language].link}">TIP</a>
				{/if}

						<div class="game-book"><a class="lnk-book" target="_blank"><img class="img-book" width="24" height="22" src="https://www.betarena.com/wp-content/uploads/2021/04/betano_v2.svg" alt="Icon Betano"></a></div>
						<div class="game-result"><div class="result first loser">{g.localScore}</div><div class="result second ">{g.visitorScore}</div></div>
					</div> 
				{/each}
			
		</div>  
		
		</div>
				
		
		{/each}
	</div>
	{/await}

</div>

<!-- ===============
  COMPONENT STYLE
==================== -->
<style>

.game-table-container {
	width: 100% !important;
	background: #FFFFFF;
	box-shadow: 0px 4px 16px rgba(0, 0, 0, 0.08);
	border-radius: 12px;
}

	.game-table-container .base {
		background: transparent !important;
		width: 100% !important;
		height: auto !important;
	}

	.game-table-container .day {
		padding: 7px 0 !important;
		width: 55px !important;
	}

		.game-table-container .day div:nth-child(1) {
			font-size: 14px;
			color: #292929;
			text-align: center;
		}

		.game-table-container .day div:nth-child(2) {
			font-weight: 500 !important;
			font-size: 14px !important;
			color: #292929 !important;
			text-align: center;
		}

		.game-table-container .day.current {
			background: transparent !important;
		}

		.game-table-container .day.selected {
			background: #F5620F !important;
			box-shadow: 0px 3px 8px rgba(212, 84, 12, 0.32) !important;
			border-radius: 6px !important;
		}

		.game-table-container .day.current div:nth-child(1), .game-table-container .day.current div:nth-child(2) {
			color: #F5620F !important;
		}

		.game-table-container .day.selected div:nth-child(1), .game-table-container .day.selected div:nth-child(2) {
			color: #fff !important;
		}

	.game-table-container .dayList {
		display: flex !important;
		justify-content: space-evenly !important;
		padding: 20px;
	}

	.game-table-container .day:hover {
		background: #F2F2F2 !important;
		border-radius: 6px !important;
	}

		.game-table-container .day:hover div:nth-child(1), .game-table-container .day:hover div:nth-child(2) {
			color: #292929 !important;
		}

	.game-table-container .day.selected:hover {
		background: #F5620F !important;
		box-shadow: 0px 3px 8px rgba(212, 84, 12, 0.32) !important;
		border-radius: 6px !important;
	}

		.game-table-container .day.selected:hover div:nth-child(1), .game-table-container .day.selected:hover div:nth-child(2) {
			color: #fff !important;
		}

	.game-table-container .betarena-menu {
		text-align: left !important;
		width: 100% !important;
		margin: 0 !important;
		border-bottom: 1px solid #E6E6E6 !important;
		display: flex;
		padding: 0 20px;
		margin-bottom: 30px !important;
	}

		.game-table-container .betarena-menu span, .game-table-container .betarena-menu span {
			width: 50%;
			font-weight: 500;
			font-size: 14px;
			color: #8C8C8C;
			font-family: "Roboto", Sans-serif !important;
			text-align: center;
			padding: 3px 0 !important;
			height: auto !important;
			position: relative;
		}

			.game-table-container .betarena-menu span.lnkSelected, .game-table-container .betarena-menu span.lnkSelected span {
				color: #F5620F !important;
			}

				.game-table-container .betarena-menu span.lnkSelected:after {
					content: '';
					position: absolute;
					display: block;
					width: 100%;
					height: 1px;
					background: #F5620F;
					bottom: -1px;
				}

		.game-table-container .betarena-menu .numLive:after {
			content: '';
			position: absolute;
			background-image: url("assets/images/live.svg");
			width: 16px;
			height: 16px;
			right: -22px;
			top: -7px;
		}

.leaguesList {
	padding: 0 20px !important;
}

.league-name {
	color: #292929 !important;
	font-size: 16px !important;
	text-transform: capitalize !important;
	font-weight: 500 !important;
}

.icon-league {
	display: flex;
}

.full-time .game-time {
	color: #8C8C8C;
	font-size: 14px;
}



.league-body {
	display: flex;
	align-items: center;
}

	.league-body .game-teams {
		flex-grow: 1;
	}

	.league-body .game-tip {
		color: #292929 !important;
		font-size: 12px !important;
		font-weight: 500 !important;
		padding: 6px 12px !important;
		height: auto !important;
		width: auto !important;
		border: 1px solid #CCCCCC !important;
		box-sizing: border-box !important;
		backdrop-filter: blur(20px) !important;
		border-radius: 4px !important;
		background: transparent !important;
	}

		.league-body .game-tip:hover {
			color: #F5620F !important;
			border: 1px solid #F5620F !important;
		}

	.league-body .lnk-book {
		display: flex;
	}

	.league-body .game-result {
		display: flex;
		flex-direction: column;
	}

.game-result .loser {
	font-weight: 500 !important;
	font-size: 14px !important;
	text-align: center !important;
	color: #8C8C8C !important;
	font-family: "Roboto", Sans-serif !important;
}

.game-result .result:not(.loser) {
	font-weight: 500 !important;
	font-size: 14px !important;
	text-align: center !important;
	color: #292929 !important;
	font-family: "Roboto", Sans-serif !important;
}

.in-progress .game-result .game-time, .in-progress .game-result .result {
	font-weight: 500 !important;
	font-size: 14px !important;
	text-align: center !important;
	color: #FF0A0A !important;
	font-family: "Roboto", Sans-serif !important;
}

.leaguesList {
	padding-bottom: 12px !important;
}

.more-games {
	border-top: 1px solid #EBEBEB !important;
	background: transparent !important;
	padding: 25px 0 !important;
	font-weight: 500 !important;
	font-size: 14px !important;
	color: #F5620F !important;
}

.mainFrame *, .game-table-container {
	font-family: 'Roboto'
}

.teamname {
    vertical-align: middle;
}

.cards {
	vertical-align: middle;
}

.three {
    height: 24px;
}

.nogames {
	text-align: center;
	margin-top: 10px;
}

.game-list {
    padding-bottom: 3px;
}

.nogames1 {
	font-size: 13px;
}

.nogames2,.nogames3{
	text-align: center;
	font-size:12px;
	color:#A1A1A1;
	margin-top: 10px;
}

.nogames span{
	color:black;
	cursor: pointer;
}



.tabela-de-jogos {	height: 19px;	width: 128px;	color: #434343;		font-size: 16px;	font-weight: bold;	letter-spacing: 0.25px;	line-height: 19px;}

.base {    height: 48px;
	width: 357px;
	box-sizing: content-box;
	margin-top: 20px;
	margin:0 auto;
}

.league:not(:first-child){
	border-top: 1px solid #EFF0F1;
	margin-top: 5px;
}

.league-title{
	margin-bottom: 10px;
	margin-top: 15px;
}

.league-title img{
	vertical-align: middle;
}

.league-body {
	margin-bottom: 1px;
	min-height: 45px;
	font-size: 0;
	padding-top: 3px;
	/*padding-bottom:7px;
		min-height: 45px;	*/
}
/*
.elementor-shortcode{
    padding-bottom: 25px;
	width:100%;
	height: auto;
	margin:0 auto;
}*/

.league-name {
	color: #292929;
	font-size: 16px;
	font-weight: 500;
	margin-left: 19px;
	line-height: 150%;
	font-style: normal;
    margin-top: 3px;
}

.result {
	height:15px;
	line-height: 17px;
	width: 100%;
	font-size: 12px;
	letter-spacing: -0.07px;
	text-align: right;
	display: inline-block;
	overflow: hidden;
	margin-top: 3px;
	margin-bottom: 4px;
}

.game-teams,.game-book,.game-result,.game-col-1,.table-game-tip,.game-media
{
	display:inline-block;
}

.game-col-1{
	vertical-align:middle;
	padding-right:11px;
}

a.table-game-tip {
	width: 43px;
	height: 30px;
	background: #FFFFFF;
	vertical-align: middle;
	text-align: center;
	font-weight: bold;
	margin-right: 15px;
	text-decoration: none;
	font-size: 12px;
	letter-spacing: -0.07px;
	line-height: 30px;
	border: 1px solid #CCCCCC;
	color: #292929;
	border-radius: 6px;
}

	a.table-game-tip:hover {
		border: 1px solid #F5620F;
		color: #F5620F;
		font-size: 12px !important;
        line-height: 30px;
        font-weight: bold !important;
	}

.game-time, .game-status {
	width: 37px;
	text-align: left;
	vertical-align: middle;
	/*font-family: "Rubik Regular";*/
	height: 21px;
	color: #292929;
	font-size: 14px;
	letter-spacing: -0.07px;
	line-height: 21px;
	text-transform: capitalize;
}

.game-table-container .game-media {
	width: 30px;
	height: 30px;
	text-align: left;
	vertical-align: middle;
	color: #FF9D09;
	font-size: 15px;
	letter-spacing: -0.07px;
	line-height: 30px;
	text-transform: capitalize;
	vertical-align: middle;
	margin-right: 10px;
}

.game-media{
	height:30px;
	width:30px;
}

	.game-media img {
        border: 2px solid #CCCCCC;
        border-radius: 16px;
        padding: 5px;
	}


.game-status {
	color: #DB0000;
	text-transform: capitalize;
	text-align:center!important;
}

.game-teams
{
	width: 182px;
	border-left:1px solid #eee;
	vertical-align: middle;
	padding-left: 9px;
}

	.game-teams .team {
		height: 20px;
		width: 100%;
		color: #292929;
		font-size: 14px;
		font-weight: 500;
		line-height: 17px;
		margin-top: 2px;
		margin-left: 7px !important;
        font-family: "Roboto", Sans-serif !important;
	}

		.game-teams .team.second{
			position:relative;
		}

		.game-book {
			vertical-align: middle;
			width: 30px;
		}

.game-result
{
	width: 30px;
	margin-left:15px;
	text-align: right;
	vertical-align: middle;
}

.img-flag {
	width: 24px;
    height: 18px;
	margin-left: 6px;
	filter: drop-shadow(0px 2px 3px rgba(0, 0, 0, 0.1));
	border-radius: 2px !important;
    vertical-align: middle!important;
}

.full-time .game-time{
	color:gray;
}


.redcard{
	margin-left: 11px;
	/*width:8px;
	height: 11px;
	background: #B92429;*/
	display: inline-block;
	vertical-align:-3px!important;
}


.lnkAll {
	height: 35px;
	width: 50%;
	color: #8C8C8C;
	font-family: "Roboto";
	font-size: 14px;
	font-weight: 500;
	line-height: 35px;
	cursor: pointer;
	display: inline-block;
	text-align: center;
}

.lnkLive {
	cursor: pointer;
	line-height:35px;
	/*height: 35px;
	width: 50%;
	color: #8C8C8C;
	font-family: "Roboto";
	font-size: 14px;
	font-weight: 500;
	line-height: 35px;
	display: inline-block;
	text-align: center;*/
}

.lnkSelected {
	color: #F5620F;
	border-bottom: 1px solid #F5620F;
}

.team.loser {
	color: #8C8C8C!important;
}
.full-time .result:not(.loser) {
	color: #292929;
}

.in-progress .game-time,.in-progress .result{
	color:red!important;
}


.in-progress .game-time {
	padding-left: 8px;
	padding-top: 10px;
}


.base {
	color: #292929;
}

.dayList .day {
	width: 46px !important;
	height: 56px !important;
	display: inline-block;
	font-size: 14px;
	cursor: pointer;
	height: 100%;
	padding:7px 0 !important;
	font-weight: normal;
}

	.dayList .day:hover {
		background:#F2F2F2;
		border-radius: 6px;
	}
	.dayList .day.current {
		color: #F5620F !important;
	}

	.dayList .day.sel {
		background: #F5620F !important;
		box-shadow: 0px 3px 8px rgba(212, 84, 12, 0.32);
		border-radius: 6px;
		color: white !important;
	}

    .game-table-container .day.sel div:nth-child(1) {
	    color: white !important;
    }

.day div {
    height: 21px!important;
}

.dayList {
	text-align: center;
	width: 100%;
	display: inline-block;
}

.not-started .game-result{
	display:none!important;
}

.dayNumber{
	font-weight: normal;
	padding-top:3px;
}

.lnk-book img.img-book {
	width: 30px;
	height: 30px;
    border-radius: 4px;
}

.more-games {
	width: 100%;
	background: white;
	padding: 5px;
	text-align: center;
	cursor: pointer;
	color: #F5620F;
	font-size: 14px;
	border-top: 1px solid #EBEBEB;
}

.lnk-book{
	vertical-align: 	middle;	
}

.league:first-child .league-title{
	margin-top: 0px!important;
}


@keyframes clockwise {
	to {transform: rotate(360deg) translatez(0);}
}

@keyframes counter-clockwise {
	to {transform: rotate(-360deg) translatez(0);}
}

@keyframes bounce {
	50% {transform: translatey(-20px);}
	100% {transform: translatey( 20px);}
}

@keyframes zoom {
	to {
		width: calc(250px + 20px);
		margin-left: calc(-125px - 10px);
		margin-top: calc(-125px - 10px);
		border-width: 10px;
		border-color: hsla(0, 0%, 78%, 1);
	}
}

@keyframes follow {
	0% { transform: translatex(-45px); }
	100% { transform: translatex( 60px); }
}


.two {
	height: 50px;
	width:  50px;
	border-width: 5px;
	border-style: solid;
	border-color: hsla(0, 0%, 78%, 0.75)
	hsla(0, 0%, 78%, 0.75) hsla(0, 0%, 78%, 0.25) hsla(0, 0%, 78%, 0.25);
	border-radius: 100%;
	animation: clockwise .5s linear infinite;
	margin:0 auto;
}

.game-teams{
	text-decoration: none;
}

.leaguesList{
	/*width:357px;*/
	width:100%;
	margin:0 auto;
}


.betarena-menu{
	width: 99%;
	margin: 0 auto;
	margin-top: 10px;
	margin-bottom: 10px;
}

.game-table-container{
	width: 357px;
	margin: 0 auto;
}

.game-table-container .betarena-menu {
    margin-bottom: 15px !important;
}

.game-table-container .dayList {
    padding-bottom: 5px;
}

@media only screen and (max-device-width: 780px) {
    .leaguesList {
        width: 100%;
        margin: 0 auto;
        padding: 0px 12px!important;
    }

    .league-name {
		margin-left: 13px

    }

	.game-teams {
		width: 162px;
	}

		.game-teams .team {
            position: relative;
            text-overflow: ellipsis;
            max-width: 140px;
            overflow: hidden;
            white-space: nowrap;
		}

	a.table-game-tip {
		width: 32px;
		height: 20px;
		font-weight: normal;
		line-height: 20px!important;
        margin-right: 5px;
        font-size: 10px !important;
        color: #292929;
    }

    .game-teams {
        padding-left: 5px;
    }


	.game-table-container .dayList {
		padding: 10px;
	}

    .game-teams .team {
        margin-top: 2px;
        margin-bottom: 2px;
    }

	.game-col-1 {
        padding-right: 6px!important;
    }

    .game-result {
        margin-left: 0px!important;
        width: 26px !important;
    }

 

    .league-body {
        margin-bottom: 0px;
        padding-bottom: 0px;
        padding-top: 0px;
    }

    .league:not(:first-child) {
        margin-top: 0px;
    }

    .lnk-book img.img-book {
        height: 20px;
        width: 20px;
    }

	.game-media img {
		max-width: 15px;
		border: 1px solid #CCCCCC;
        border-radius: 16px;
        padding: 3px;
    }

    .game-table-container .game-media {
        margin-right: 0px!important;
    }

    .game-book {
        width: 25px;
    }

    .result {
        height: 17px;
    }

	.league {
        padding-bottom: 7px;
    }

    .game-table-container .game-media {
        width: 21px !important;
    }

    .result {
        margin-bottom: 2px!important;
        margin-top: 1px!important;
    }
}

.game-table-container .day.sel div:nth-child(2) {
    color: white !important;
}


[game-status="FT"] .game-col-1 .game-status, [game-status="FT"] .game-col-1 .game-time {
	color: #8C8C8C !important;
}
</style>