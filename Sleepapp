<script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, doc, setDoc, deleteDoc, onSnapshot, updateDoc, getDoc, arrayUnion } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // --- App State ---
        let state = {
            activeTab: 'organizer',
            user: null,
            isAuthReady: false,
            teamFeather: [],
            teamFang: [],
            teamTrials: [],
            mentors: [],
            settings: {
                api: { clientId: '179eaf3a69454a35824e37a301ffc151', clientSecret: 'qNKUNKJlAU8nMx2PElgRNH9bACNHw7vO' },
                guild: { name: 'Cult Of Sleep', realm: 'Turalyon', region: 'eu', officers: 'Officer1,Officer2' }
            },
            searchedCharacter: null,
            guildRoster: {
                members: [],
                isLoading: false,
                loadingMessage: '',
                error: null,
                searchTerm: '',
                selectedClass: 'All',
                selectedRole: 'All',
                hasLoadedOnce: false // To track initial load
            },
            search: {
                characterName: '',
                realm: 'Turalyon',
                region: 'eu',
                isLoading: false,
                error: null,
            }
        };

        // --- Firebase Configuration & Initialization ---
        const firebaseConfig = {
  apiKey: "AIzaSyDVWcNNv2jfee6JmycN5HXEmKrQ9x-0J3c",
  authDomain: "cultofsleep-6cdb7.firebaseapp.com",
  projectId: "cultofsleep-6cdb7",
  storageBucket: "cultofsleep-6cdb7.firebasestorage.app",
  messagingSenderId: "55622141495",
  appId: "1:55622141495:web:c31a083a723e564c1561b2"
};
        const appId = 'default-cult-of-sleep';
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);

        // --- Helper Functions ---
        const handleImgError = (e, name) => {
            e.target.onerror = null;
            e.target.src = `https://placehold.co/64x64/1F2937/EAB308?text=${name.charAt(0)}`;
        };
        const handleItemIconError = (e) => {
            e.target.onerror = null;
            e.target.style.display = 'none';
        };
        const getClassColor = (className) => ({'Death Knight': '#C41F3B', 'Demon Hunter': '#A330C9', 'Druid': '#FF7D0A', 'Evoker': '#33937F', 'Hunter': '#A8D469', 'Mage': '#3FC7EB', 'Monk': '#00FF96', 'Paladin': '#F48CBA', 'Priest': '#FFFFFF', 'Rogue': '#FFF468', 'Shaman': '#0070DD', 'Warlock': '#8788EE', 'Warrior': '#C69B6D'}[className] || '#FFFFFF');
        const getRoleFromSpec = (specName) => ({'Blood': 'Tank', 'Vengeance': 'Tank', 'Guardian': 'Tank', 'Brewmaster': 'Tank', 'Protection': 'Tank', 'Restoration': 'Healer', 'Preservation': 'Healer', 'Mistweaver': 'Healer', 'Holy': 'Healer', 'Discipline': 'Healer', 'Frost': 'DPS', 'Unholy': 'DPS', 'Havoc': 'DPS', 'Balance': 'DPS', 'Feral': 'DPS', 'Devastation': 'DPS', 'Augmentation': 'DPS', 'Beast Mastery': 'DPS', 'Marksmanship': 'DPS', 'Survival': 'DPS', 'Arcane': 'DPS', 'Fire': 'DPS', 'Windwalker': 'DPS', 'Retribution': 'DPS', 'Shadow': 'DPS', 'Assassination': 'DPS', 'Outlaw': 'DPS', 'Subtlety': 'DPS', 'Elemental': 'DPS', 'Enhancement': 'DPS', 'Affliction': 'DPS', 'Demonology': 'DPS', 'Destruction': 'DPS', 'Arms': 'DPS', 'Fury': 'DPS'}[specName] || 'DPS');
        const getMythicPlusColor = (score) => {
            if (score >= 2500) return 'text-orange-400';
            if (score >= 2000) return 'text-purple-400';
            if (score >= 1500) return 'text-blue-400';
            if (score >= 750) return 'text-green-400';
            return 'text-gray-400';
        };
        const getWarcraftLogsUrl = (char, guildConfig) => {
            if (!char || !guildConfig.region) return '#';
            const realmSlug = String(char.realm?.slug || char.realm).toLowerCase().replace(/\s+/g, '-');
            const charName = String(char.name).toLowerCase();
            const region = guildConfig.region;
            return `https://www.warcraftlogs.com/character/${region}/${realmSlug}/${charName}`;
        };
        
        const calculateIlvlGain = (history) => {
            if (!history || history.length < 1) return { gain: 0, since: null };
            const thirtyDaysAgo = new Date();
            thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

            const relevantHistory = history
                .map(h => ({ ...h, date: new Date(h.date) }))
                .filter(h => h.date >= thirtyDaysAgo)
                .sort((a, b) => a.date - b.date);

            if (relevantHistory.length < 1) return { gain: 0, since: null };

            const initialEntry = relevantHistory[0];
            const latestEntry = relevantHistory[relevantHistory.length - 1];
            
            const gain = latestEntry.itemLevel - initialEntry.itemLevel;
            
            return { gain: gain, since: initialEntry.date };
        };


        // --- Notification Logic ---
        const showNotification = (message, type = 'success') => {
            const notificationId = `notif-${Date.now()}`;
            const container = document.getElementById('notification-container');
            const notification = document.createElement('div');
            const baseClasses = "fixed bottom-5 right-5 p-4 rounded-lg shadow-xl text-white transition-opacity duration-300 z-50";
            const typeClasses = { success: "bg-green-600", error: "bg-red-600" };
            notification.className = `${baseClasses} ${typeClasses[type] || 'bg-gray-700'}`;
            notification.id = notificationId;
            notification.textContent = message;
            container.appendChild(notification);
            setTimeout(() => {
                const el = document.getElementById(notificationId);
                if (el) el.remove();
            }, 4000);
        };
        
        // --- Templating / Component Functions ---
        const LinkIcon = () => '<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" /></svg>';
        const RefreshIcon = () => '<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 4v5h5M20 20v-5h-5M20 4h-5v5M4 20h5v-5M12 4v2.25c-2.43.5-4.5 2.57-5 5H4.75c.5-3.58 3.42-6.5 7-7v2.25L16 8l-4-4zM12 20v-2.25c2.43-.5 4.5-2.57 5-5h2.25c-.5 3.58-3.42 6.5-7 7v-2.25L8 16l4 4z"/></svg>';
        const CheckIcon = () => `<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-green-400" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" /></svg>`;
        
        const RoleIcon = (role) => {
            const icons = {
                Tank: `<svg xmlns="http://www.w3.org/2000/svg" title="Tank" class="h-4 w-4 text-blue-400" viewBox="0 0 24 24" fill="currentColor"><path d="M12 1L3 5v6c0 5.55 3.84 10.74 9 12 5.16-1.26 9-6.45 9-12V5l-9-4z"></path></svg>`,
                Healer: `<svg xmlns="http://www.w3.org/2000/svg" title="Healer" class="h-4 w-4 text-green-400" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" /></svg>`,
                DPS: `<svg xmlns="http://www.w3.org/2000/svg" title="DPS" class="h-4 w-4 text-red-500" viewBox="0 0 24 24" fill="currentColor"><path d="M16.8,3.23,15.42,4.62,18,7.21l1.39-1.39a1.5,1.5,0,0,0,0-2.12,1.5,1.5,0,0,0-2.12,0ZM14,6,4.5,15.5,3,17l2,2,1.5-1.5L16,8ZM2,22l5-5-2-2Z"></path></svg>`
            };
            return icons[role] || '';
        };

        const Header = () => {
            const guildName = state.settings.guild.name.replace(/-/g, ' ').replace(/\b\w/g, l => l.toUpperCase());
            return `
                <header class="text-center mb-4">
                    <h1 class="text-4xl md:text-5xl font-bold text-yellow-400 tracking-wider">${guildName}</h1>
                    <h2 class="text-2xl md:text-3xl font-bold text-gray-300">Raid Organizer</h2>
                </header>
            `;
        };

        const TabNavigation = () => {
            const tabs = ['organizer', 'roster', 'trials', 'mentors', 'settings'];
            return `
                <div class="flex justify-center border-b border-gray-700 mb-6">
                    ${tabs.map(tab => `
                        <button data-tab="${tab}" class="tab-button py-3 px-6 font-semibold text-lg transition-colors duration-300 capitalize ${state.activeTab === tab ? 'text-yellow-400 border-b-2 border-yellow-400' : 'text-gray-400 hover:text-yellow-300'}">
                            ${tab.replace('-', ' ')}
                        </button>
                    `).join('')}
                </div>
            `;
        };

        const PlayerHeader = (player, guildConfig, onClick = '') => {
            const classColor = getClassColor(player.class);
            const mythicPlusScore = player.mythicPlusScore ?? player.mythic_plus_scores_by_season?.[0]?.scores?.all ?? 0;
            const scoreColor = getMythicPlusColor(mythicPlusScore);
            const raidProgression = player.raidProgression ?? player.raid_progression;
            const ilvlGainData = calculateIlvlGain(player.itemLevelHistory);
            const role = getRoleFromSpec(player.spec);
            
            let ilvlGainText = '';
            if (ilvlGainData.since) {
                const formattedDate = ilvlGainData.since.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
                ilvlGainText = `(+${ilvlGainData.gain.toFixed(2)} iLvl since ${formattedDate})`;
            }

            return `
                <div class="flex items-center flex-grow min-w-0 ${onClick ? 'cursor-pointer' : ''}" ${onClick ? `onclick="${onClick}"` : ''}>
                    <img src="${player.thumbnail_url}" alt="${player.name}" class="w-12 h-12 md:w-16 md:h-16 rounded-full border-2 flex-shrink-0" style="border-color: ${classColor}" onerror="this.onerror=null; this.src='https://placehold.co/64x64/1F2937/EAB308?text=${player.name.charAt(0)}';">
                    <div class="ml-3 min-w-0">
                        <h4 class="font-bold text-lg flex items-center gap-2 truncate" style="color: ${classColor};">
                            ${RoleIcon(role)}
                            <span class="truncate">${player.name}</span>
                            <a href="${getWarcraftLogsUrl(player, guildConfig)}" onclick="event.stopPropagation()" target="_blank" rel="noopener noreferrer" title="View Warcraft Logs" class="text-gray-400 hover:text-white flex-shrink-0">
                                ${LinkIcon()}
                            </a>
                        </h4>
                        <p class="text-sm text-gray-300 truncate">${player.race} ${player.spec} ${player.class}</p>
                        <div class="flex items-center gap-3 text-sm flex-wrap">
                            <p class="font-semibold text-green-400">iLvl: ${player.itemLevel}</p>
                            ${ilvlGainText ? `<p class="font-semibold ${ilvlGainData.gain > 0 ? 'text-yellow-400' : 'text-gray-400'} text-xs">${ilvlGainText}</p>` : ''}
                            <p class="font-bold ${scoreColor}">M+: ${Math.round(mythicPlusScore)}</p>
                        </div>
                        ${raidProgression ? RaidProgress(raidProgression, 'the-war-within-raid-1', 'TWW Prog') : ''}
                        ${player.startDate ? `<p class="text-xs text-gray-400 italic pt-1">Trial Started: ${player.startDate}</p>` : ''}
                        ${player.mentorName ? `<p class="text-xs text-yellow-300 italic pt-1">Assigned to mentor - ${player.mentorName}</p>` : ''}
                    </div>
                </div>
            `;
        };

        const RaidProgress = (raidProgression, raidName, raidLabel) => {
            const raidData = raidProgression ? raidProgression[raidName] : null;
            if (!raidData) return `<div class="text-xs mt-1"><p class="font-semibold text-gray-300">${raidLabel}: <span class="text-gray-400 font-normal">N/A</span></p></div>`;
            return `
                <div class="text-xs mt-1">
                    <p class="font-semibold text-gray-300">${raidLabel}:
                        <span class="font-normal text-blue-400 ml-2">H: ${raidData.heroic_bosses_killed}/${raidData.total_bosses}</span>
                        <span class="font-normal text-purple-400 ml-2">M: ${raidData.mythic_bosses_killed}/${raidData.total_bosses}</span>
                    </p>
                </div>
            `;
        };
        
        const ItemList = (items) => {
            const slotOrder = ['head', 'neck', 'shoulder', 'back', 'chest', 'wrist', 'hands', 'waist', 'legs', 'feet', 'finger1', 'finger2', 'trinket1', 'trinket2', 'mainhand', 'offhand'];
            const equippedItems = slotOrder.map(slot => ({ slot, ...items[slot] })).filter(item => items[item.slot]);
            return `
                <div class="bg-gray-800 p-4 animate-fade-in-down">
                    <h5 class="text-lg font-semibold mb-3 text-gray-300 border-b border-gray-700 pb-2">Equipped Gear</h5>
                    <ul class="space-y-2">
                        ${equippedItems.map(item => `
                            <li class="flex items-center bg-gray-700/50 p-2 rounded-md">
                                <a href="https://www.wowhead.com/item=${item.item_id}" target="_blank" rel="noopener noreferrer">
                                    <img src="https://wow.zamimg.com/images/wow/icons/large/${item.icon}.jpg" alt="${item.name}" class="w-10 h-10 rounded border-2" style="border-color: var(--quality-${item.quality})" onerror="this.onerror=null; this.style.display='none';">
                                </a>
                                <div class="ml-3 flex-grow">
                                    <a href="https://www.wowhead.com/item=${item.item_id}" target="_blank" rel="noopener noreferrer" class="font-semibold hover:underline" style="color: var(--quality-${item.quality})">${item.name}</a>
                                    <p class="text-xs text-gray-400">${item.slot.charAt(0).toUpperCase() + item.slot.slice(1).replace(/(\\d)/, ' $1')}</p>
                                </div>
                                <div class="text-right"><p class="font-bold text-lg text-gray-200">${item.item_level}</p></div>
                            </li>
                        `).join('')}
                    </ul>
                </div>
            `;
        };

        const BossKills = (raidProgression) => {
            const raid = raidProgression['the-war-within-raid-1'];
            if (!raid) return '';

            const difficulties = [
                { name: 'Mythic', kills: raid.mythic_bosses_killed_details, color: 'text-purple-400' },
                { name: 'Heroic', kills: raid.heroic_bosses_killed_details, color: 'text-blue-400' },
                { name: 'Normal', kills: raid.normal_bosses_killed_details, color: 'text-green-400' },
            ];

            return `
                <div class="bg-gray-800 p-4 mt-2 animate-fade-in-down">
                    <h5 class="text-lg font-semibold mb-3 text-gray-300 border-b border-gray-700 pb-2">Boss Kills (The War Within)</h5>
                    <div class="space-y-3">
                    ${difficulties.map(diff => {
                        if (!diff.kills || diff.kills.length === 0) return '';
                        return `
                            <div>
                                <h6 class="font-bold ${diff.color}">${diff.name} (${diff.kills.length}/${raid.total_bosses})</h6>
                                <ul class="grid grid-cols-2 gap-x-4 gap-y-1 mt-1">
                                    ${diff.kills.map(boss => `
                                        <li class="flex items-center text-sm text-gray-200">
                                            ${CheckIcon()}
                                            <span class="ml-2">${boss.name}</span>
                                        </li>
                                    `).join('')}
                                </ul>
                            </div>
                        `;
                    }).join('')}
                    </div>
                </div>
            `;
        };

        const PlayerCard = (player, teamId) => {
            const safePlayerId = player.id.replace(/'/g, "\\'");
            const trialInfo = state.teamTrials.find(t => t.id === player.id);
            let mentorName = null;
            if (trialInfo && trialInfo.mentor) {
                const mentor = state.mentors.find(m => m.id === trialInfo.mentor);
                if (mentor) {
                    mentorName = mentor.name;
                }
            }
            const playerForHeader = { ...player, mentorName: mentorName };

            return `
                <div id="player-card-${player.id}" class="bg-gray-700 rounded-lg shadow-md transition-all duration-300 player-card">
                    <div class="p-3 flex items-center justify-between">
                        <div class="flex-grow min-w-0 p-2" draggable="true" data-player-id="${player.id}" data-team-id="${teamId}">
                             ${PlayerHeader(playerForHeader, state.settings.guild, `window.togglePlayerCard('${safePlayerId}')`)}
                        </div>
                        <div class="flex flex-col items-center space-y-2 ml-2">
                            ${teamId === 'teamFang' ? `
                                <button data-move-to-trials="${player.id}" data-team-name="${teamId}" data-player-name="${player.name}" title="Move to Trials" class="bg-yellow-600 hover:bg-yellow-700 text-white font-bold py-1 px-2 rounded text-xs transition">To Trials</button>
                            ` : '<div class="h-6"></div>' }
                            <div class="flex">
                                <button data-refresh-player="${player.id}" data-team-name="${teamId}" title="Refresh Player Data" class="refresh-btn text-gray-400 hover:text-white p-1">${RefreshIcon()}</button>
                                <button data-remove-player="${player.id}" data-team-name="${teamId}" data-player-name="${player.name}" class="text-gray-400 hover:text-red-500 font-bold text-xl px-2 z-10">&times;</button>
                            </div>
                        </div>
                    </div>
                    <div id="item-list-${player.id}" class="hidden rounded-b-lg">
                        ${player.gear && player.gear.items ? ItemList(player.gear.items) : ''}
                        ${player.raidProgression ? BossKills(player.raidProgression) : ''}
                    </div>
                </div>
            `;
        };
        
        const RoleDisplay = (team) => {
            const roleCounts = team.reduce((acc, p) => {
                const pRole = getRoleFromSpec(p.spec);
                acc[pRole] = (acc[pRole] || 0) + 1;
                return acc;
            }, { Tank: 0, Healer: 0, DPS: 0 });

            return `
                <div class="bg-gray-900/50 rounded-md p-2 mb-4">
                    <div class="flex justify-around text-sm text-gray-300 font-semibold">
                        <span>Total: <span class="text-white">${team.length}</span>/20</span>
                        <span>Tanks: <span class="text-white">${roleCounts.Tank}</span>/2</span>
                        <span>Healers: <span class="text-white">${roleCounts.Healer}</span>/4</span>
                        <span>DPS: <span class="text-white">${roleCounts.DPS}</span>/14</span>
                    </div>
                </div>
            `;
        };

        const TeamColumn = (title, team, teamId) => `
            <div class="bg-gray-800 rounded-lg shadow-lg p-6 min-h-[300px] drop-zone" data-team-id="${teamId}">
                <h3 class="text-3xl font-bold mb-2 text-center text-yellow-400 border-b-2 border-gray-700 pb-2">${title}</h3>
                ${RoleDisplay(team)}
                <div class="space-y-3">
                    ${team.length > 0 ? team.map(player => PlayerCard(player, teamId)).join('') : '<p class="text-gray-500 text-center pt-8">Drag players here or use search to add them.</p>'}
                </div>
            </div>
        `;

        const SearchForm = () => `
            <form id="search-form" class="bg-gray-800 p-6 rounded-lg shadow-lg mb-4 flex flex-col md:flex-row items-center justify-center gap-4">
                <input type="text" id="search-char-name" value="${state.search.characterName}" placeholder="Character Name" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400"/>
                <input type="text" id="search-char-realm" value="${state.search.realm}" placeholder="Realm" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400" disabled/>
                <select id="search-char-region" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400" disabled>
                    <option value="eu">EU</option>
                </select>
                <button type="submit" ${state.search.isLoading ? 'disabled' : ''} class="bg-yellow-500 hover:bg-yellow-600 text-gray-900 font-bold w-full md:w-auto py-3 px-6 rounded-md transition duration-300 disabled:bg-gray-500">
                    ${state.search.isLoading ? '...' : 'Search'}
                </button>
            </form>
        `;

        const SearchResults = () => {
            if (!state.searchedCharacter) return '';
            const character = state.searchedCharacter;
            return `
                <div class="bg-gray-800 rounded-lg shadow-lg p-4 my-4 flex flex-col md:flex-row items-center justify-between">
                    ${PlayerHeader({ ...character, spec: character.active_spec_name, itemLevel: character.gear.item_level_equipped }, state.settings.guild)}
                    <div class="flex flex-col md:flex-row gap-2 mt-4 md:mt-0">
                        <button class="add-player-button bg-yellow-600 hover:bg-yellow-700 text-white font-bold py-2 px-4 rounded-lg transition" data-team-name="trials">Add to Trials</button>
                        <button class="add-player-button bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition" data-team-name="teamFeather">Add to Feather</button>
                        <button class="add-player-button bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg transition" data-team-name="teamFang">Add to Fang</button>
                    </div>
                </div>
            `;
        };
        
        const RaidOrganizerPage = () => `
            <div class="animate-fade-in">
                ${SearchForm()}
                ${state.search.isLoading ? '<div class="text-center p-4">Searching...</div>' : ''}
                ${state.search.error ? `<div class="text-center p-4 bg-red-800 rounded-lg my-4 cursor-pointer" onclick="window.clearSearchError()">${state.search.error}</div>` : ''}
                ${SearchResults()}
                <div class="mt-8 grid grid-cols-1 lg:grid-cols-2 gap-8">
                    ${TeamColumn("Feather", state.teamFeather, "teamFeather")}
                    ${TeamColumn("Fang", state.teamFang, "teamFang")}
                </div>
            </div>
        `;

        const SettingsPage = () => `
            <div class="bg-gray-800 rounded-lg shadow-lg p-6 animate-fade-in max-w-2xl mx-auto">
                <h3 class="text-3xl font-bold mb-6 text-center text-yellow-400 border-b-2 border-gray-700 pb-3">Settings</h3>
                <form id="settings-form">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                        <div>
                            <h4 class="text-2xl font-bold mb-4 text-yellow-300">API Settings</h4>
                            <p class="text-gray-400 mb-4">API credentials are now hardcoded.</p>
                             <div class="p-3 bg-gray-700 rounded-md">
                                 <span class="block text-sm font-medium text-gray-300 mb-1">Client ID</span>
                                 <span class="text-white font-semibold break-all">${state.settings.api.clientId}</span>
                             </div>
                        </div>
                        <div>
                            <h4 class="text-2xl font-bold mb-4 text-yellow-300">Guild Settings</h4>
                            <p class="text-gray-400 mb-4">Your guild's information.</p>
                            <div class="space-y-4">
                                <div class="p-3 bg-gray-700 rounded-md">
                                    <span class="block text-sm font-medium text-gray-300 mb-1">Guild Name</span>
                                    <span class="text-white font-semibold">${state.settings.guild.name}</span>
                                </div>
                                <div class="p-3 bg-gray-700 rounded-md">
                                    <span class="block text-sm font-medium text-gray-300 mb-1">Realm</span>
                                    <span class="text-white font-semibold">${state.settings.guild.realm}</span>
                                </div>
                                <div class="p-3 bg-gray-700 rounded-md">
                                    <span class="block text-sm font-medium text-gray-300 mb-1">Region</span>
                                    <span class="text-white font-semibold">${state.settings.guild.region.toUpperCase()}</span>
                                </div>
                                <div><label for="officers" class="block text-sm font-medium text-gray-300 mb-1">Officers (comma-separated)</label><input type="text" id="officers" name="officers" placeholder="Officer1,Officer2" value="${state.settings.guild.officers || ''}" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400" /></div>
                            </div>
                        </div>
                    </div>
                    <div class="mt-8 text-center"><button type="submit" class="bg-yellow-500 hover:bg-yellow-600 text-gray-900 font-bold py-3 px-8 rounded-md transition duration-300">Save Officer List</button></div>
                </form>
            </div>
        `;
        
        const PlayerTrialsPage = () => {
            if (!state.teamTrials.length) return `<p class="text-gray-400 text-center py-8">There are no players currently on trial.</p>`;
            return `
                <div class="bg-gray-800 rounded-lg shadow-lg p-6 animate-fade-in">
                    <h3 class="text-3xl font-bold mb-4 text-center text-yellow-400 border-b-2 border-gray-700 pb-2">Player Trials</h3>
                    <div class="grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 gap-6">
                        ${state.teamTrials.map(TrialPlayerCard).join('')}
                    </div>
                </div>
            `;
        };

        const TrialPlayerCard = (trial) => {
            const { player, mentor, checklist, notes = '', startDate } = trial;
            const assignedMentor = state.mentors.find(m => m.id === mentor);
            const playerForHeader = { 
                ...player, 
                spec: player.active_spec_name, 
                itemLevel: player.gear.item_level_equipped, 
                startDate: startDate,
                mentorName: assignedMentor ? assignedMentor.name : null
            };

            return `
                <div class="bg-gray-700 rounded-lg p-4 flex flex-col gap-4 shadow-md">
                    ${PlayerHeader(playerForHeader, state.settings.guild)}
                    <div>
                        <label for="mentor-${trial.id}" class="block text-sm font-medium text-gray-300 mb-1">Mentor</label>
                        <select id="mentor-${trial.id}" data-trial-id="${trial.id}" class="trial-mentor-select bg-gray-600 text-white w-full p-2 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400">
                            <option value="">Unassigned</option>
                            ${state.mentors.map(m => `<option value="${m.id}" ${mentor === m.id ? 'selected' : ''}>${m.name}</option>`).join('')}
                        </select>
                    </div>
                    <div>
                        <h5 class="text-md font-medium text-gray-300 mb-2">Trial Checklist</h5>
                        <div class="space-y-2">
                            ${Object.entries(checklist).map(([task, isDone]) => `
                                <label class="flex items-center space-x-3 cursor-pointer">
                                    <input type="checkbox" ${isDone ? 'checked' : ''} data-trial-id="${trial.id}" data-task="${task}" class="trial-checklist-item h-5 w-5 rounded bg-gray-600 border-gray-500 text-yellow-500 focus:ring-yellow-400"/>
                                    <span class="${isDone ? 'text-green-400 line-through' : 'text-gray-200'}">${task}</span>
                                </label>
                            `).join('')}
                        </div>
                    </div>
                    <div>
                        <label for="notes-${trial.id}" class="block text-sm font-medium text-gray-300 mb-1">Notes</label>
                        <textarea id="notes-${trial.id}" data-trial-id="${trial.id}" rows="4" class="trial-notes-area bg-gray-600 text-white w-full p-2 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400 resize-y" placeholder="Add trial notes here...">${notes}</textarea>
                    </div>
                    <div class="flex gap-2 w-full mt-2">
                        <button class="promote-button flex-1 bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-3 rounded text-sm transition" data-trial-id="${trial.id}" data-team-name="teamFeather">Promote to Feather</button>
                        <button class="promote-button flex-1 bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-3 rounded text-sm transition" data-trial-id="${trial.id}" data-team-name="teamFang">Promote to Fang</button>
                        <button class="remove-trial-button bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-3 rounded text-sm transition" data-trial-id="${trial.id}" data-player-name="${player.name}">Remove</button>
                    </div>
                </div>
            `;
        };
        
        const MentorsPage = () => {
            const raiders = [...state.teamFeather, ...state.teamFang];
            
            return `
                <div class="animate-fade-in">
                    <div class="bg-gray-800 rounded-lg shadow-lg p-6 mb-8">
                        <h3 class="text-3xl font-bold mb-4 text-center text-yellow-400 border-b-2 border-gray-700 pb-2">Manage Mentors</h3>
                        <div class="flex flex-col md:flex-row gap-4 items-center">
                            <select id="mentor-select" class="bg-gray-700 text-white w-full md:w-1/2 p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400">
                                <option value="">Select a raider to make a mentor...</option>
                                ${raiders.map(r => `<option value="${r.id}">${r.name}</option>`).join('')}
                            </select>
                            <button id="add-mentor-btn" class="bg-green-600 hover:bg-green-700 text-white font-bold w-full md:w-auto py-3 px-6 rounded-md transition duration-300">Add Mentor</button>
                        </div>
                    </div>

                    <div class="bg-gray-800 rounded-lg shadow-lg p-6">
                         <h3 class="text-3xl font-bold mb-4 text-center text-yellow-400 border-b-2 border-gray-700 pb-2">Mentor Assignments</h3>
                         <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                            ${state.mentors.map(mentor => `
                                <div class="bg-gray-900/50 rounded-lg p-4 min-h-[150px]">
                                    <div class="flex justify-between items-center mb-4">
                                        <h4 class="font-bold text-xl" style="color: ${getClassColor(mentor.class)}">${mentor.name}</h4>
                                        <button data-mentor-id="${mentor.id}" class="remove-mentor-btn text-red-500 hover:text-red-400 font-bold text-2xl">&times;</button>
                                    </div>
                                    <div class="space-y-2">
                                    ${state.teamTrials.filter(t => t.mentor === mentor.id).map(MentorTrialCard).join('')}
                                    </div>
                                </div>
                            `).join('')}
                         </div>
                    </div>
                </div>
            `;
        };

        const MentorTrialCard = (trial) => {
            const player = trial.player;
            const assignedMentor = state.mentors.find(m => m.id === trial.mentor);
            const playerForHeader = {
                ...player,
                spec: player.active_spec_name || player.spec,
                itemLevel: player.gear?.item_level_equipped || player.itemLevel,
                itemLevelHistory: player.itemLevelHistory || [],
                mentorName: assignedMentor ? assignedMentor.name : null
            };
            const safeTrialId = trial.id.replace(/'/g, "\\'");
            return `
                <div id="mentor-trial-card-${trial.id}" class="bg-gray-700 rounded-lg shadow-md transition-all duration-300 player-card">
                    <div class="p-3">
                        ${PlayerHeader(playerForHeader, state.settings.guild, `window.toggleMentorTrialCard('${safeTrialId}')`)}
                    </div>
                    <div id="mentor-item-list-${trial.id}" class="hidden">
                        ${player.gear && player.gear.items ? ItemList(player.gear.items) : ''}
                        ${player.raidProgression ? BossKills(player.raidProgression) : ''}
                    </div>
                </div>
            `;
        };


        const GuildRosterPage = () => {
            const { members, isLoading, error, searchTerm, selectedClass, selectedRole } = state.guildRoster;
            const classes = ['All', 'Death Knight', 'Demon Hunter', 'Druid', 'Evoker', 'Hunter', 'Mage', 'Monk', 'Paladin', 'Priest', 'Rogue', 'Shaman', 'Warlock', 'Warrior'];
            const roles = ['All', 'Tank', 'Healer', 'DPS'];
            
            const filteredMembers = members.filter(member => {
                if (!member.raiderioData) return false;

                const charName = member.blizzardData.name.toLowerCase();
                const className = member.raiderioData.class;
                const role = getRoleFromSpec(member.raiderioData.active_spec_name);

                const nameMatch = charName.includes(searchTerm.toLowerCase());
                const classMatch = selectedClass === 'All' || className === selectedClass;
                const roleMatch = selectedRole === 'All' || role === selectedRole;

                return nameMatch && classMatch && roleMatch;
            });

            return `
                <div class="bg-gray-800 rounded-lg shadow-lg p-6 animate-fade-in">
                    <div class="flex justify-between items-center border-b-2 border-gray-700 pb-2 mb-4">
                        <h3 class="text-3xl font-bold text-yellow-400">Guild Roster</h3>
                        <button id="refresh-roster-btn" ${isLoading ? 'disabled' : ''} class="bg-blue-600 hover:bg-blue-500 text-white font-bold py-2 px-4 rounded transition disabled:bg-gray-500 disabled:cursor-not-allowed">
                            ${isLoading ? 'Loading...' : 'Refresh Roster'}
                        </button>
                    </div>
                    <div class="mb-4 space-y-4">
                        <div>
                             <label for="roster-search-term" class="block text-sm font-medium text-gray-300 mb-1">Search by Name</label>
                             <input type="text" id="roster-search-term" placeholder="Character name..." value="${searchTerm}" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400" />
                        </div>
                        <div class="flex flex-col md:flex-row gap-4">
                            <div class="w-full">
                                <label for="roster-class-filter-select" class="block text-sm font-medium text-gray-300 mb-1">Filter by Class</label>
                                <select id="roster-class-filter-select" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400">
                                     ${classes.map(className => `<option value="${className}" ${selectedClass === className ? 'selected' : ''}>${className}</option>`).join('')}
                                </select>
                            </div>
                            <div class="w-full">
                                <label for="roster-role-filter-select" class="block text-sm font-medium text-gray-300 mb-1">Filter by Role</label>
                                <select id="roster-role-filter-select" class="bg-gray-700 text-white w-full p-3 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-400">
                                     ${roles.map(role => `<option value="${role}" ${selectedRole === role ? 'selected' : ''}>${role}</option>`).join('')}
                                </select>
                            </div>
                        </div>
                    </div>
                    ${error ? `<p class="text-red-400 text-center py-4">Error: ${error}</p>` : ''}
                    ${(isLoading && !state.guildRoster.hasLoadedOnce) ? '' : 
                        filteredMembers.length > 0 ? `
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4">
                            ${filteredMembers.map(RosterMemberCard).join('')}
                        </div>`
                    : `<p class="text-gray-400 text-center py-8">No members found. Check settings or filters.</p>`}
                </div>
            `;
        };

        const RosterMemberCard = (member) => {
            const { blizzardData, raiderioData, detailError } = member;
            
            const realmSlug = member.blizzardData.realm.slug;
            const playerId = `${member.blizzardData.name}-${realmSlug}`;
            const isInFang = state.teamFang.some(p => p.id === playerId);
            
            const trialInfo = state.teamTrials.find(t => t.id === playerId);
            let mentorName = null;
            if (trialInfo && trialInfo.mentor) {
                const mentor = state.mentors.find(m => m.id === trialInfo.mentor);
                if (mentor) {
                    mentorName = mentor.name;
                }
            }

            const playerForHeader = raiderioData ? { 
                ...raiderioData, 
                spec: raiderioData.active_spec_name, 
                itemLevel: raiderioData.gear.item_level_equipped,
                mentorName: mentorName
            } : { 
                name: blizzardData.name, 
                class: blizzardData.playable_class.name, 
                race: blizzardData.playable_race.name, 
                spec: 'Loading...', 
                itemLevel: '...', 
                thumbnail_url: '', 
                realm: blizzardData.realm.slug,
                mentorName: mentorName
            };
            
            return `
                <div id="roster-card-${blizzardData.id}" class="bg-gray-700 rounded-lg shadow-md transition-all duration-300 flex flex-col">
                    <div class="p-3 flex flex-col gap-3 flex-grow">
                        ${PlayerHeader(playerForHeader, state.settings.guild, `window.toggleRosterCard('${blizzardData.id}')`)}
                        ${detailError ? '<p class="text-xs text-red-400 text-center">Could not load details.</p>' : ''}
                        ${raiderioData ? `
                            <div class="flex flex-col gap-2 w-full mt-auto pt-2">
                                ${isInFang ? `
                                    <button class="add-roster-player-button w-full bg-yellow-600 hover:bg-yellow-700 text-white font-bold py-1 px-2 rounded text-sm transition" data-char-id="${blizzardData.id}" data-team-name="trials">Add to Trials</button>
                                ` : ''}
                                <div class="flex gap-2 w-full">
                                    <button class="add-roster-player-button flex-1 bg-blue-600 hover:bg-blue-700 text-white font-bold py-1 px-2 rounded text-sm transition" data-char-id="${blizzardData.id}" data-team-name="teamFeather">To Feather</button>
                                    <button class="add-roster-player-button flex-1 bg-green-600 hover:bg-green-700 text-white font-bold py-1 px-2 rounded text-sm transition" data-char-id="${blizzardData.id}" data-team-name="teamFang">To Fang</button>
                                </div>
                            </div>
                        ` : ''}
                    </div>
                    <div id="roster-item-list-${blizzardData.id}" class="hidden">
                        ${raiderioData?.gear ? ItemList(raiderioData.gear.items) : ''}
                        ${raiderioData?.raid_progression ? BossKills(raiderioData.raid_progression) : ''}
                    </div>
                </div>
            `;
        };

        // --- Main Render Function ---
        const render = () => {
            const appContainer = document.getElementById('app');
            let currentPageHtml = '';
            switch (state.activeTab) {
                case 'roster': currentPageHtml = GuildRosterPage(); break;
                case 'trials': currentPageHtml = PlayerTrialsPage(); break;
                case 'mentors': currentPageHtml = MentorsPage(); break;
                case 'settings': currentPageHtml = SettingsPage(); break;
                case 'organizer':
                default: currentPageHtml = RaidOrganizerPage(); break;
            }
            
            appContainer.innerHTML = `
                <div class="container mx-auto">
                    ${Header()}
                    ${TabNavigation()}
                    <div class="mt-6">${currentPageHtml}</div>
                </div>
            `;
            attachEventListeners();
        };

        // --- Event Listeners ---
        const attachEventListeners = () => {
            // Tab navigation
            document.querySelectorAll('.tab-button').forEach(button => {
                button.addEventListener('click', (e) => {
                    const newTab = e.currentTarget.dataset.tab;
                    if (newTab === 'roster' && !state.guildRoster.hasLoadedOnce) {
                        fetchGuildRoster();
                    }
                    state.activeTab = newTab;
                    render();
                });
            });

            // Page-specific listeners
            if (state.activeTab === 'organizer') {
                document.getElementById('search-form')?.addEventListener('submit', handlePlayerSearch);
                document.getElementById('search-char-name')?.addEventListener('input', e => state.search.characterName = e.target.value);
                
                document.querySelectorAll('.add-player-button').forEach(btn => btn.addEventListener('click', handleAddSearchedPlayer));
                
                // Drag and Drop for Organizer
                document.querySelectorAll('[draggable="true"]').forEach(card => card.addEventListener('dragstart', handleDragStart));
                document.querySelectorAll('.drop-zone').forEach(zone => {
                    zone.addEventListener('dragover', handleDragOver);
                    zone.addEventListener('drop', handleDrop);
                });
                document.querySelectorAll('[data-remove-player]').forEach(btn => btn.addEventListener('click', handleRemovePlayer));
                document.querySelectorAll('[data-refresh-player]').forEach(btn => btn.addEventListener('click', handleRefreshPlayer));
                document.querySelectorAll('[data-move-to-trials]').forEach(btn => btn.addEventListener('click', handleMoveToTrials));
            }

            if (state.activeTab === 'settings') {
                document.getElementById('settings-form')?.addEventListener('submit', handleSaveSettings);
            }

            if (state.activeTab === 'trials') {
                document.querySelectorAll('.trial-mentor-select').forEach(sel => sel.addEventListener('change', handleUpdateTrial));
                document.querySelectorAll('.trial-checklist-item').forEach(chk => chk.addEventListener('change', handleUpdateTrial));
                document.querySelectorAll('.trial-notes-area').forEach(area => area.addEventListener('blur', handleUpdateTrial));
                document.querySelectorAll('.promote-button').forEach(btn => btn.addEventListener('click', handlePromoteTrial));
                document.querySelectorAll('.remove-trial-button').forEach(btn => btn.addEventListener('click', handleRemoveTrial));
            }
            
             if (state.activeTab === 'mentors') {
                document.getElementById('add-mentor-btn')?.addEventListener('click', handleAddMentor);
                document.querySelectorAll('.remove-mentor-btn').forEach(btn => btn.addEventListener('click', handleRemoveMentor));
            }
            
            if (state.activeTab === 'roster') {
                document.getElementById('refresh-roster-btn')?.addEventListener('click', fetchGuildRoster);
                document.getElementById('roster-search-term')?.addEventListener('input', e => {
                    state.guildRoster.searchTerm = e.target.value;
                    render();
                });
                document.getElementById('roster-class-filter-select')?.addEventListener('change', e => {
                    state.guildRoster.selectedClass = e.target.value;
                    render();
                });
                document.getElementById('roster-role-filter-select')?.addEventListener('change', e => {
                    state.guildRoster.selectedRole = e.target.value;
                    render();
                });
                document.querySelectorAll('.add-roster-player-button').forEach(btn => btn.addEventListener('click', handleAddFromRoster));
            }
        };

        // --- Event Handlers & Logic ---
        window.togglePlayerCard = (playerId) => {
            const itemList = document.getElementById(`item-list-${playerId}`);
            if (itemList) itemList.classList.toggle('hidden');
        };
        window.toggleRosterCard = (charId) => {
            const itemList = document.getElementById(`roster-item-list-${charId}`);
            if (itemList) itemList.classList.toggle('hidden');
        };
        window.toggleMentorTrialCard = (trialId) => {
            const itemList = document.getElementById(`mentor-item-list-${trialId}`);
            if (itemList) itemList.classList.toggle('hidden');
        };
        window.clearSearchError = () => {
            state.search.error = null;
            render();
        };

        const handlePlayerSearch = async (e) => {
            e.preventDefault();
            if (!state.search.characterName) {
                state.search.error = 'Please enter a character name.';
                render();
                return;
            }
            state.search.isLoading = true;
            state.search.error = null;
            state.searchedCharacter = null;
            render();

            try {
                const { characterName, realm, region } = state.search;
                const url = `https://raider.io/api/v1/characters/profile?region=${region}&realm=${realm}&name=${encodeURIComponent(characterName)}&fields=gear,mythic_plus_scores_by_season:current,raid_progression`;
                const response = await fetch(url);
                if (!response.ok) throw new Error('Character not found on Raider.io.');
                const data = await response.json();
                state.searchedCharacter = data;
            } catch (err) {
                state.search.error = err.message;
            } finally {
                state.search.isLoading = false;
                render();
            }
        };

        const addPlayerToTeam = async (teamName, player) => {
            if (!state.user) {
                showNotification("You must be logged in to modify teams.", "error");
                return false;
            }

            const realmSlug = player.realm?.slug || player.realm;
            if (!realmSlug || typeof realmSlug !== 'string') {
                console.error("Invalid realm data for player:", player);
                showNotification("Failed to add player due to invalid realm data.", "error");
                return false;
            }

            const team = teamName === 'teamFeather' ? state.teamFeather : state.teamFang;
            const role = getRoleFromSpec(player.active_spec_name || player.spec);
            const roleCounts = team.reduce((acc, p) => {
                const pRole = getRoleFromSpec(p.spec);
                acc[pRole] = (acc[pRole] || 0) + 1;
                return acc;
            }, { Tank: 0, Healer: 0, DPS: 0 });

            if (team.length >= 20) { showNotification("Team is full (20/20 players).", "error"); return false; }
            if (role === 'Tank' && roleCounts.Tank >= 2) { showNotification("Tank slots are full (2/2).", "error"); return false; }
            if (role === 'Healer' && roleCounts.Healer >= 4) { showNotification("Healer slots are full (4/4).", "error"); return false; }
            if (role === 'DPS' && roleCounts.DPS >= 14) { showNotification("DPS slots are full (14/14).", "error"); return false; }

            const uniquePlayerId = `${player.name}-${realmSlug}`;
            const playerDocRef = doc(db, "artifacts", appId, "public", "data", teamName, uniquePlayerId);
            
            const itemLevelHistory = player.itemLevelHistory || [{ date: new Date().toISOString(), itemLevel: player.gear.item_level_equipped }];

            const playerData = {
                name: player.name, 
                realm: realmSlug,
                region: player.region, 
                class: player.class,
                race: player.race, 
                spec: player.active_spec_name || player.spec, 
                itemLevel: player.gear.item_level_equipped,
                thumbnail_url: player.thumbnail_url,
                mythicPlusScore: player.mythic_plus_scores_by_season?.[0]?.scores?.all || 0,
                raidProgression: player.raid_progression,
                gear: player.gear,
                itemLevelHistory: itemLevelHistory
            };
            try {
                await setDoc(playerDocRef, playerData);
                showNotification(`${player.name} added to ${teamName === 'teamFeather' ? 'Feather' : 'Fang'}.`);
                return true;
            } catch (e) {
                console.error("Error adding player:", e);
                showNotification("Failed to add player.", "error");
                return false;
            }
        };

        const addPlayerToTrials = async (player) => {
            if (!state.user) { showNotification("You must be logged in.", "error"); return; }
            
            const realmSlug = player.realm?.slug || player.realm;
            if (!realmSlug || typeof realmSlug !== 'string') {
                console.error("Invalid realm data for trial player:", player);
                showNotification("Failed to add trial due to invalid realm data.", "error");
                return;
            }

            const uniquePlayerId = `${player.name}-${realmSlug}`;
            const playerDocRef = doc(db, "artifacts", appId, "public", "data", "teamTrials", uniquePlayerId);
            const trialData = {
                player: {
                    ...player,
                    itemLevelHistory: [{ date: new Date().toISOString(), itemLevel: player.gear.item_level_equipped }]
                }, 
                mentor: '',
                checklist: { "Attended 2 Raids": false, "Discord Setup Correctly": false, "Positive Attitude": false, "Reviewed Logs": false },
                notes: '',
                startDate: new Date().toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' })
            };
            try {
                await setDoc(playerDocRef, trialData);
                showNotification(`${player.name} added to trials.`);
            } catch (e) { console.error("Error adding player to trials:", e); showNotification("Failed to add player to trials.", "error"); }
        };

        const removePlayer = async (teamName, playerId, playerName) => {
            if (!state.user) return;
            try {
                await deleteDoc(doc(db, "artifacts", appId, "public", "data", teamName, playerId));
                showNotification(`${playerName} removed.`);
            } catch (e) { console.error("Error removing player:", e); showNotification("Failed to remove player.", "error"); }
        };
        
        const handleMoveToTrials = (e) => {
            e.stopPropagation();
            const { moveToTrials: playerId, teamName } = e.currentTarget.dataset;
            const sourceTeamList = teamName === 'teamFeather' ? state.teamFeather : state.teamFang;
            const player = sourceTeamList.find(p => p.id === playerId);

            if (player) {
                addPlayerToTrials(player);
                // The line to remove the player is now gone
            }
        };

        const handleAddSearchedPlayer = (e) => {
            const teamName = e.currentTarget.dataset.teamName;
            if (teamName === 'trials') {
                addPlayerToTrials(state.searchedCharacter);
            } else {
                addPlayerToTeam(teamName, state.searchedCharacter);
            }
            state.searchedCharacter = null;
            render();
        };

        const handleRemovePlayer = (e) => {
            e.stopPropagation();
            const { removePlayer: playerId, teamName, playerName } = e.currentTarget.dataset;
            removePlayer(teamName, playerId, playerName);
        };
        
        const handleRefreshPlayer = async (e) => {
            e.stopPropagation();
            const { refreshPlayer: playerId, teamName } = e.currentTarget.dataset;
            const team = [...state.teamFeather, ...state.teamFang, ...state.teamTrials.map(t => ({...t.player, id: t.id}))];
            const player = team.find(p => p.id === playerId);

            if (!player) return;

            showNotification(`Refreshing ${player.name}...`);
            try {
                const url = `https://raider.io/api/v1/characters/profile?region=${player.region}&realm=${player.realm}&name=${encodeURIComponent(player.name)}&fields=gear,mythic_plus_scores_by_season:current,raid_progression`;
                const response = await fetch(url);
                if (!response.ok) throw new Error('Character not found on Raider.io.');
                const data = await response.json();
                
                const playerDocRef = doc(db, "artifacts", appId, "public", "data", teamName, playerId);
                
                const updatedData = {
                    gear: data.gear,
                    itemLevel: data.gear.item_level_equipped,
                    mythicPlusScore: data.mythic_plus_scores_by_season?.[0]?.scores?.all || 0,
                    raidProgression: data.raid_progression,
                    itemLevelHistory: arrayUnion({ date: new Date().toISOString(), itemLevel: data.gear.item_level_equipped })
                };

                if (teamName.startsWith('team')) {
                    await updateDoc(playerDocRef, updatedData);
                } else if (teamName === 'teamTrials') {
                     await updateDoc(playerDocRef, { player: { ...player, ...updatedData } });
                }

                showNotification(`${player.name} updated successfully!`);
            } catch (err) {
                console.error("Failed to refresh player:", err);
                showNotification(`Failed to refresh ${player.name}.`, "error");
            }
        };


        const handleSaveSettings = async (e) => {
            e.preventDefault();
            if (!state.user) return;
            const formData = new FormData(e.target);
            const newSettings = {
                guild: { 
                    ...state.settings.guild,
                    officers: formData.get('officers'),
                }
            };
            const guildSettingsRef = doc(db, "artifacts", appId, "public", "data", "app_settings", "guild_config");
            try {
                await setDoc(guildSettingsRef, newSettings.guild, { merge: true }); 
                showNotification("Officer list saved successfully!");
            } catch (err) { console.error("Error saving settings:", err); showNotification("Failed to save settings.", "error"); }
        };

        // --- Drag and Drop Handlers ---
        let draggedPlayer = null;
        let sourceTeam = null;

        const handleDragStart = (e) => {
            const playerId = e.currentTarget.dataset.playerId;
            const teamId = e.currentTarget.dataset.teamId;
            const team = teamId === 'teamFeather' ? state.teamFeather : state.teamFang;
            draggedPlayer = team.find(p => p.id === playerId);
            sourceTeam = teamId;
            e.dataTransfer.effectAllowed = 'move';
        };

        const handleDragOver = (e) => e.preventDefault();

        const handleDrop = async (e) => {
            e.preventDefault();
            const targetTeam = e.currentTarget.dataset.teamId;
            if (sourceTeam !== targetTeam && draggedPlayer) {
                const wasAdded = await addPlayerToTeam(targetTeam, draggedPlayer);
                if (wasAdded) {
                    await removePlayer(sourceTeam, draggedPlayer.id, draggedPlayer.name);
                }
            }
            draggedPlayer = null;
            sourceTeam = null;
        };
        
        // --- Trial Handlers ---
        const handleUpdateTrial = async (e) => {
            if (!state.user) return;
            const { trialId } = e.currentTarget.dataset;
            const playerDocRef = doc(db, "artifacts", appId, "public", "data", "teamTrials", trialId);
            let updatedData = {};

            if (e.currentTarget.matches('.trial-mentor-select')) {
                updatedData = { mentor: e.target.value };
            } else if (e.currentTarget.matches('.trial-checklist-item')) {
                const task = e.currentTarget.dataset.task;
                const trial = state.teamTrials.find(t => t.id === trialId);
                updatedData = { checklist: { ...trial.checklist, [task]: e.target.checked } };
            } else if (e.currentTarget.matches('.trial-notes-area')) {
                updatedData = { notes: e.target.value };
            }
            
            try { 
                await updateDoc(playerDocRef, updatedData); 
                if(updatedData.mentor !== undefined) showNotification("Mentor assigned.");
            }
            catch (err) { console.error("Error updating trial player:", err); showNotification("Failed to update trial.", "error"); }
        };

        const handlePromoteTrial = (e) => {
            const { trialId, teamName } = e.currentTarget.dataset;
            const trial = state.teamTrials.find(t => t.id === trialId);
            if (trial) {
                addPlayerToTeam(teamName, trial.player);
                removePlayer('teamTrials', trial.id, trial.player.name);
            }
        };

        const handleRemoveTrial = (e) => {
            const { trialId, playerName } = e.currentTarget.dataset;
            removePlayer('teamTrials', trialId, playerName);
        };
        
        // --- Mentor Handlers ---
        const handleAddMentor = async () => {
            const select = document.getElementById('mentor-select');
            const playerId = select.value;
            if (!playerId) return;

            const raider = [...state.teamFeather, ...state.teamFang].find(r => r.id === playerId);
            if (raider && !state.mentors.find(m => m.id === playerId)) {
                const mentorDocRef = doc(db, "artifacts", appId, "public", "data", "mentors", raider.id);
                try {
                    await setDoc(mentorDocRef, { name: raider.name, class: raider.class });
                    showNotification(`${raider.name} is now a mentor.`);
                } catch (e) {
                    console.error("Error adding mentor:", e);
                    showNotification("Failed to add mentor.", "error");
                }
            }
        };

        const handleRemoveMentor = async (e) => {
            const { mentorId } = e.currentTarget.dataset;
            const mentor = state.mentors.find(m => m.id === mentorId);
            if (!mentor) return;
            
            const mentorDocRef = doc(db, "artifacts", appId, "public", "data", "mentors", mentorId);
            try {
                await deleteDoc(mentorDocRef);
                // Unassign trials from this mentor
                state.teamTrials.forEach(trial => {
                    if (trial.mentor === mentorId) {
                        const trialDocRef = doc(db, "artifacts", appId, "public", "data", "teamTrials", trial.id);
                        updateDoc(trialDocRef, { mentor: "" });
                    }
                });
                showNotification(`${mentor.name} is no longer a mentor.`);
            } catch (e) {
                console.error("Error removing mentor:", e);
                showNotification("Failed to remove mentor.", "error");
            }
        };

        // --- Roster Handlers ---
        let jokeInterval;
        const fetchGuildRoster = async () => {
            if (state.guildRoster.isLoading) return;
            if (!state.settings.api.clientId || !state.settings.api.clientSecret) { 
                state.guildRoster.error = "Blizzard API credentials are not set."; 
                render(); 
                return; 
            }
            
            state.guildRoster.isLoading = true;
            state.guildRoster.error = null;
            
            let showLoadingScreen = !state.guildRoster.hasLoadedOnce;
            if (showLoadingScreen) {
                const loadingScreen = document.getElementById('loading-screen');
                const loadingBar = document.getElementById('loading-bar');
                const jokeEl = document.getElementById('loading-joke');
                loadingScreen.classList.remove('hidden');
                loadingBar.style.width = '0%';
                
                const jokes = [
                    "You are seeing this loading screen because Terevious codes even worse than he plays World of Warcraft.",
                    "Terevious's code is like a Mythic+ affix: unnecessarily complicated and full of bugs.",
                    "Fetching data... probably faster than Terevious can find his keybinds.",
                    "Compiling spaghetti code... Terevious's favorite recipe.",
                    "If this takes too long, it's because Terevious is probably standing in the fire again.",
                    "Debugging Terevious's logic... this might take a while."
                ];
                let jokeIndex = 0;
                jokeEl.textContent = jokes[jokeIndex];
                jokeInterval = setInterval(() => {
                    jokeIndex = (jokeIndex + 1) % jokes.length;
                    jokeEl.textContent = jokes[jokeIndex];
                }, 3000);

            } else {
                render();
            }
            
            try {
                const tokenResponse = await fetch('https://oauth.battle.net/token', {
                    method: 'POST', body: 'grant_type=client_credentials',
                    headers: { 'Authorization': 'Basic ' + btoa(`${state.settings.api.clientId}:${state.settings.api.clientSecret}`), 'Content-Type': 'application/x-www-form-urlencoded' }
                });
                if (!tokenResponse.ok) throw new Error(`Token Error: ${tokenResponse.status}`);
                const tokenData = await tokenResponse.json();
                
                const { name: guildName, realm, region } = state.settings.guild;
                const rosterUrl = `https://${region}.api.blizzard.com/data/wow/guild/${realm.toLowerCase()}/${guildName.toLowerCase().replace(/\s+/g, '-')}/roster?namespace=profile-${region}&locale=en_US`;
                const rosterResponse = await fetch(rosterUrl, { headers: { 'Authorization': `Bearer ${tokenData.access_token}` } });
                if (!rosterResponse.ok) throw new Error(`Roster Error: ${rosterResponse.statusText}`);
                const rosterData = await rosterResponse.json();

                if (rosterData && Array.isArray(rosterData.members)) {
                    const initialMembers = rosterData.members
                        .filter(m => m.character.level === 80)
                        .map(m => ({ blizzardData: m.character, raiderioData: null, isLoadingDetails: true, detailError: false }));
                    state.guildRoster.members = initialMembers;
                    
                    let loadedCount = 0;
                    const totalMembers = initialMembers.length;

                    if (totalMembers === 0) {
                        if (showLoadingScreen) {
                             document.getElementById('loading-bar').style.width = '100%';
                             clearInterval(jokeInterval);
                             setTimeout(() => document.getElementById('loading-screen').classList.add('hidden'), 500);
                        }
                    }

                    const promises = initialMembers.map(member => 
                        fetchRaiderIOData(member.blizzardData, region).finally(() => {
                            loadedCount++;
                            if (showLoadingScreen) {
                                const percent = (loadedCount / totalMembers) * 100;
                                document.getElementById('loading-bar').style.width = `${percent}%`;
                            }
                        })
                    );
                    await Promise.all(promises);

                } else { 
                    state.guildRoster.members = []; 
                }
                state.guildRoster.hasLoadedOnce = true;
            } catch (err) { 
                state.guildRoster.error = err.message; 
            } finally { 
                state.guildRoster.isLoading = false; 
                if (showLoadingScreen) {
                    clearInterval(jokeInterval);
                    setTimeout(() => document.getElementById('loading-screen').classList.add('hidden'), 500);
                }
                render(); 
            }
        };

        const fetchRaiderIOData = async (blizzardCharacter, region) => {
            try {
                const url = `https://raider.io/api/v1/characters/profile?region=${region}&realm=${blizzardCharacter.realm.slug}&name=${encodeURIComponent(blizzardCharacter.name.toLowerCase())}&fields=gear,mythic_plus_scores_by_season:current,raid_progression`;
                const res = await fetch(url);
                if (!res.ok) throw new Error("Raider.io fetch failed");
                const detailedData = await res.json();
                
                state.guildRoster.members = state.guildRoster.members.map(m => 
                    m.blizzardData.id === blizzardCharacter.id ? { ...m, raiderioData: detailedData, isLoadingDetails: false } : m
                );
            } catch(err) {
                console.error(`Failed to load details for ${blizzardCharacter.name}`, err);
                 state.guildRoster.members = state.guildRoster.members.map(m => 
                    m.blizzardData.id === blizzardCharacter.id ? { ...m, isLoadingDetails: false, detailError: true } : m
                );
            }
        };

        const handleAddFromRoster = (e) => {
            const { charId, teamName } = e.currentTarget.dataset;
            const member = state.guildRoster.members.find(m => m.blizzardData.id == charId);
            if (member && member.raiderioData) {
                if (teamName === 'trials') {
                    addPlayerToTrials(member.raiderioData);
                } else {
                    addPlayerToTeam(teamName, member.raiderioData);
                }
            } else {
                showNotification("Character data is not fully loaded yet.", "error");
            }
        };
        
        // --- Initial Load & Firebase Listeners ---
        const authSignIn = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (e) {
                console.error("Authentication failed:", e);
                showNotification("Authentication failed.", "error");
            }
        };

        onAuthStateChanged(auth, (currentUser) => {
            state.user = currentUser;
            state.isAuthReady = true;

            // Once authenticated, set up listeners for data that can change
            const settingsGuildRef = doc(db, "artifacts", appId, "public", "data", "app_settings", "guild_config");
            onSnapshot(settingsGuildRef, docSnap => {
                if (docSnap.exists()) {
                    // Only update officers, keep the rest hardcoded
                    state.settings.guild.officers = docSnap.data().officers || state.settings.guild.officers;
                }
                render();
            });

            const featherColRef = collection(db, "artifacts", appId, "public", "data", "teamFeather");
            onSnapshot(featherColRef, (snapshot) => {
                state.teamFeather = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                render();
            });
            
            const fangColRef = collection(db, "artifacts", appId, "public", "data", "teamFang");
            onSnapshot(fangColRef, (snapshot) => {
                state.teamFang = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                render();
            });
            
            const trialsColRef = collection(db, "artifacts", appId, "public", "data", "teamTrials");
            onSnapshot(trialsColRef, (snapshot) => {
                state.teamTrials = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                render();
            });

            const mentorsColRef = collection(db, "artifacts", appId, "public", "data", "mentors");
            onSnapshot(mentorsColRef, (snapshot) => {
                state.mentors = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                render();
            });
        });

        // Initial call
        authSignIn();
        render();
    </script>
