<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delayed Odds Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #121212;
            color: #e0e0e0;
        }
        .container {
            max-width: 900px;
        }
        .card {
            background-color: #1e1e1e;
            border: 1px solid #333;
            transition: transform 0.2s ease-in-out;
        }
        .card:hover {
            transform: translateY(-5px);
        }
        .odds-row {
            border-top: 1px solid #333;
        }
        .loading-spinner {
            border: 4px solid rgba(255, 255, 255, 0.2);
            border-left-color: #fff;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="p-4 sm:p-8">

    <div class="container mx-auto">
        <header class="text-center mb-8">
            <h1 class="text-3xl sm:text-4xl font-bold mb-2">Sports Odds Tracker</h1>
            <p class="text-gray-400">
                Live odds from major US sportsbooks. Data may be slightly delayed for informational purposes.
            </p>
        </header>

        <div class="flex flex-col sm:flex-row justify-center items-center gap-4 mb-8">
            <div class="flex items-center gap-2">
                <label for="sport-select" class="text-gray-400">Filter by Sport:</label>
                <select id="sport-select" class="bg-gray-800 text-white border border-gray-600 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-green-500">
                    <option value="upcoming">All Upcoming Events</option>
                    <option value="americanfootball_nfl">NFL</option>
                    <option value="basketball_nba">NBA</option>
                    <option value="baseball_mlb">MLB</option>
                    <option value="basketball_wnba">WNBA</option>
                    <option value="americanfootball_ncaaf">College Football</option>
                    <option value="basketball_ncaab">College Basketball</option>
                    <option value="motor_nascar">Nascar</option>
                </select>
            </div>
            <div class="flex items-center gap-2">
                <label for="book-select" class="text-gray-400">Filter by Book:</label>
                <select id="book-select" class="bg-gray-800 text-white border border-gray-600 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-green-500">
                    <option value="all">All Books (Best Odds)</option>
                    <option value="draftkings">DraftKings</option>
                    <option value="fanduel">FanDuel</option>
                    <option value="betmgm">BetMGM</option>
                    <option value="betrivers">BetRivers</option>
                    <option value="espnbet">ESPN Bet</option>
                    <option value="fanatics">Fanatics</option>
                    <option value="bet365">Bet365</option>
                    <option value="ballybet">Bally Bet</option>
                    <option value="prophet">Prophet</option>
                </select>
            </div>
        </div>

        <div id="odds-container" class="space-y-6">
            <div id="loading-message" class="flex justify-center items-center py-20 flex-col">
                <div class="loading-spinner"></div>
                <p class="mt-4 text-lg">Fetching live odds...</p>
                <p class="text-yellow-400 mt-2 text-center text-sm">
                    ⚠️ The app will not work until you replace 'YOUR_API_KEY' with a real key from the-odds-api.com
                </p>
            </div>
        </div>

        <footer class="mt-12 text-center text-gray-500 text-sm">
            <p>Data provided by The Odds API. <br>For educational and personal use only. Not for real-time betting.</p>
        </footer>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // ===============================================================
            const API_KEY = "9249d264c2669f03aa6266d156012464"; 
            
            const oddsContainer = document.getElementById('odds-container');
            const loadingMessage = document.getElementById('loading-message');
            const sportSelect = document.getElementById('sport-select');
            const bookSelect = document.getElementById('book-select');

            const fetchOdds = async (sport, bookmakerFilter) => {
                loadingMessage.classList.remove('hidden');
                oddsContainer.innerHTML = ''; // Clear previous content

                const regions = "us"; 
                const markets = "h2h,spreads,totals"; 
                const oddsFormat = "american"; 

                const apiUrl = `https://api.the-odds-api.com/v4/sports/${sport}/odds/?apiKey=${API_KEY}&regions=${regions}&markets=${markets}&oddsFormat=${oddsFormat}`;

                try {
                    const response = await fetch(apiUrl);
                    if (!response.ok) {
                        throw new Error(`HTTP error! status: ${response.status}`);
                    }
                    const games = await response.json();
                    
                    loadingMessage.classList.add('hidden');
                    oddsContainer.innerHTML = ''; 

                    if (games.length === 0) {
                        oddsContainer.innerHTML = '<p class="text-center text-gray-400">No upcoming games found for this sport. Please try again later.</p>';
                        return;
                    }

                    const allowedBookmakerKeys = [
                        'fanduel', 'draftkings', 'bet365', 'betrivers', 'ballybet', 
                        'fanatics', 'espnbet', 'betmgm', 'prophet'
                    ];

                    games.forEach(game => {
                        const gameElement = document.createElement('div');
                        gameElement.className = 'card p-4 sm:p-6 rounded-lg shadow-lg';
                        
                        const homeTeam = game.home_team;
                        const awayTeam = game.away_team;
                        
                        // Filter bookmakers to include only the allowed ones
                        const filteredBookmakers = game.bookmakers.filter(bookmaker => 
                            allowedBookmakerKeys.includes(bookmaker.key)
                        );
                        
                        let bookmakersContent = '';
                        
                        if (bookmakerFilter === 'all') {
                            // Logic for finding and displaying best odds from the filtered list
                            const bestOdds = { h2h: null, spreads: null, totals: null };
                            
                            filteredBookmakers.forEach(bookmaker => {
                                // Find best H2H odds
                                const h2h = bookmaker.markets.find(m => m.key === 'h2h');
                                if (h2h) {
                                    h2h.outcomes.forEach(outcome => {
                                        if (!bestOdds.h2h || 
                                            (outcome.price > 0 && outcome.price > (bestOdds.h2h[outcome.name]?.price || -Infinity)) ||
                                            (outcome.price < 0 && (outcome.price > (bestOdds.h2h[outcome.name]?.price || -Infinity)))) {
                                            
                                            bestOdds.h2h = bestOdds.h2h || {};
                                            bestOdds.h2h[outcome.name] = { ...outcome, bookmaker: bookmaker.title };
                                        }
                                    });
                                }

                                // Find best spreads odds
                                const spreads = bookmaker.markets.find(m => m.key === 'spreads');
                                if (spreads) {
                                     spreads.outcomes.forEach(outcome => {
                                        if (!bestOdds.spreads || 
                                            (outcome.point > (bestOdds.spreads[outcome.name]?.point || -Infinity)) ||
                                            (outcome.point === (bestOdds.spreads[outcome.name]?.point) && outcome.price > (bestOdds.spreads[outcome.name]?.price || -Infinity))
                                        ) {
                                            bestOdds.spreads = bestOdds.spreads || {};
                                            bestOdds.spreads[outcome.name] = { ...outcome, bookmaker: bookmaker.title };
                                        }
                                    });
                                }

                                // Find best totals odds
                                const totals = bookmaker.markets.find(m => m.key === 'totals');
                                if (totals) {
                                    totals.outcomes.forEach(outcome => {
                                        if (!bestOdds.totals || 
                                            (outcome.point > (bestOdds.totals[outcome.name]?.point || -Infinity)) ||
                                            (outcome.point === (bestOdds.totals[outcome.name]?.point) && outcome.price > (bestOdds.totals[outcome.name]?.price || -Infinity))
                                        ) {
                                            bestOdds.totals = bestOdds.totals || {};
                                            bestOdds.totals[outcome.name] = { ...outcome, bookmaker: bookmaker.title };
                                        }
                                    });
                                }
                            });

                             // Construct content from best odds
                             bookmakersContent += `<h3 class="text-xl font-bold mt-4">Best Odds Across All Books</h3>`;
                             
                            if (bestOdds.h2h) {
                                bookmakersContent += `<div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3"><div class="col-span-2 text-sm font-semibold text-gray-300">Moneyline (H2H)</div>`;
                                Object.values(bestOdds.h2h).forEach(outcome => {
                                     bookmakersContent += `
                                        <div class="flex flex-col justify-between items-start bg-gray-700 p-2 sm:p-3 rounded-md">
                                            <span class="text-sm sm:text-base font-medium">${outcome.name === homeTeam ? 'Home' : 'Away'}</span>
                                            <span class="text-lg font-bold text-green-400">${outcome.price > 0 ? `+${outcome.price}` : outcome.price}</span>
                                            <span class="text-xs text-gray-400">${outcome.bookmaker}</span>
                                        </div>`;
                                });
                                bookmakersContent += `</div>`;
                            }

                            if (bestOdds.spreads) {
                                bookmakersContent += `<div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3"><div class="col-span-2 text-sm font-semibold text-gray-300">Spreads</div>`;
                                Object.values(bestOdds.spreads).forEach(outcome => {
                                    bookmakersContent += `
                                        <div class="flex flex-col justify-between items-start bg-gray-700 p-2 sm:p-3 rounded-md">
                                            <span class="text-sm sm:text-base font-medium">${outcome.name}</span>
                                            <span class="text-lg font-bold text-green-400">${outcome.point > 0 ? `+${outcome.point}` : outcome.point}</span>
                                            <span class="text-xs text-gray-400">${outcome.bookmaker}</span>
                                        </div>`;
                                });
                                bookmakersContent += `</div>`;
                            }

                            if (bestOdds.totals) {
                                bookmakersContent += `<div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3"><div class="col-span-2 text-sm font-semibold text-gray-300">Totals (Over/Under)</div>`;
                                Object.values(bestOdds.totals).forEach(outcome => {
                                    bookmakersContent += `
                                        <div class="flex flex-col justify-between items-start bg-gray-700 p-2 sm:p-3 rounded-md">
                                            <span class="text-sm sm:text-base font-medium">${outcome.name} ${outcome.point}</span>
                                            <span class="text-lg font-bold text-green-400">${outcome.price > 0 ? `+${outcome.price}` : outcome.price}</span>
                                            <span class="text-xs text-gray-400">${outcome.bookmaker}</span>
                                        </div>`;
                                });
                                bookmakersContent += `</div>`;
                            }

                            if (!bestOdds.h2h && !bestOdds.spreads && !bestOdds.totals) {
                                bookmakersContent = '<p class="text-gray-500 italic">No sportsbook odds available.</p>';
                            }

                        } else {
                            // Logic for displaying a single sportsbook
                            const bookmaker = filteredBookmakers.find(b => b.key === bookmakerFilter);
                            if (bookmaker) {
                                const h2hOdds = bookmaker.markets.find(m => m.key === 'h2h');
                                const spreadsOdds = bookmaker.markets.find(m => m.key === 'spreads');
                                const totalsOdds = bookmaker.markets.find(m => m.key === 'totals');

                                let marketContent = '';
                                if (h2hOdds) {
                                    marketContent += `
                                        <div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3">
                                            <div class="col-span-2 text-sm font-semibold text-gray-300">Moneyline (H2H)</div>
                                            ${h2hOdds.outcomes.map(outcome => `
                                                <div class="flex justify-between items-center bg-gray-700 p-2 sm:p-3 rounded-md">
                                                    <span class="text-sm sm:text-base font-medium">${outcome.name === homeTeam ? 'Home' : 'Away'}</span>
                                                    <span class="text-lg font-bold text-green-400">${outcome.price > 0 ? `+${outcome.price}` : outcome.price}</span>
                                                </div>
                                            `).join('')}
                                        </div>`;
                                }
                                if (spreadsOdds) {
                                    marketContent += `
                                        <div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3">
                                            <div class="col-span-2 text-sm font-semibold text-gray-300">Spreads</div>
                                            ${spreadsOdds.outcomes.map(outcome => `
                                                <div class="flex justify-between items-center bg-gray-700 p-2 sm:p-3 rounded-md">
                                                    <span class="text-sm sm:text-base font-medium">${outcome.name}</span>
                                                    <span class="text-lg font-bold text-green-400">${outcome.point > 0 ? `+${outcome.point}` : outcome.point}</span>
                                                    <span class="text-gray-400">(${outcome.price > 0 ? `+${outcome.price}` : outcome.price})</span>
                                                </div>
                                            `).join('')}
                                        </div>`;
                                }
                                if (totalsOdds) {
                                    marketContent += `
                                        <div class="odds-row grid grid-cols-2 gap-4 py-2 sm:py-3">
                                            <div class="col-span-2 text-sm font-semibold text-gray-300">Totals (Over/Under)</div>
                                            ${totalsOdds.outcomes.map(outcome => `
                                                <div class="flex justify-between items-center bg-gray-700 p-2 sm:p-3 rounded-md">
                                                    <span class="text-sm sm:text-base font-medium">${outcome.name} ${outcome.point}</span>
                                                    <span class="text-lg font-bold text-green-400">${outcome.price > 0 ? `+${outcome.price}` : outcome.price}</span>
                                                </div>
                                            `).join('')}
                                        </div>`;
                                }
                                
                                if (marketContent) {
                                     bookmakersContent += `
                                        <h3 class="text-xl font-bold mt-4">${bookmaker.title}</h3>
                                        ${marketContent}
                                    `;
                                }
                            } else {
                                bookmakersContent = '<p class="text-gray-500 italic">Odds not available for this sportsbook or the selected sport.</p>';
                            }
                        }

                        gameElement.innerHTML = `
                            <div class="flex justify-between items-center mb-4">
                                <div>
                                    <h2 class="text-xl sm:text-2xl font-bold">${homeTeam} vs ${awayTeam}</h2>
                                    <p class="text-gray-400 text-sm mt-1">${new Date(game.commence_time).toLocaleString()}</p>
                                </div>
                                <div>
                                    <span class="bg-gray-700 text-gray-300 text-xs px-2 py-1 rounded-full font-semibold">
                                        ${game.sport_title}
                                    </span>
                                </div>
                            </div>
                            ${bookmakersContent}
                        `;

                        oddsContainer.appendChild(gameElement);
                    });

                } catch (error) {
                    console.error('Failed to fetch odds:', error);
     
