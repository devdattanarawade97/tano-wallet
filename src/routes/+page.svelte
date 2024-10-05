<script>
	import { onMount } from "svelte";
	import { TonConnectUI, TonConnectUIError } from "@tonconnect/ui";

	import { PUBLIC_CREDIT_ADDRESS } from "$env/static/public";
	import { PUBLIC_BACKEND_BASE_URI } from "$env/static/public";
	import { goto } from "$app/navigation";
	/**
	 * @type {TonConnectUI}
	 */
	let tonConnectUI;
	let currentConnectWalletAddress = "";
	let chatId;

	/**
	 * @type {string | number | null}
	 */
	let charge = null;
	let telegramUsername = null;
	// console.log("base uri : ", PUBLIC_BACKEND_BASE_URI);
	//http://localhost:5173/?chat_id=5831161789&msg_text=hello&model=gpt
	onMount(() => {
		// Initialize TonConnectUI after component is mounted
		const urlParams = new URLSearchParams(window.location.search);
		chatId = urlParams.get("chat_id");
		
		charge = urlParams.get("charge");
		telegramUsername = urlParams.get("username");
		
		tonConnectUI = new TonConnectUI({
			manifestUrl: "https://tano-wallet.vercel.app/tonconnect-manifest.json",
			buttonRootId: "ton-connect",
			walletsListConfiguration: {
				includeWallets: [
					{
						appName: "tonwallet",
						name: "TON Wallet",
						imageUrl: "https://wallet.ton.org/assets/ui/qr-logo.png",
						aboutUrl:
							"https://chrome.google.com/webstore/detail/ton-wallet/nphplpgoakhhjchkkhmiggakijnkhfnd",
						universalLink: "https://wallet.ton.org/ton-connect",
						jsBridgeKey: "tonwallet",
						// bridgeUrl: "https://bridge.tonapi.io/bridge",
						bridgeUrl: "https://testnet.tonapi.io/bridge",
						platforms: ["chrome", "android"],
					},
					{
						appName: "tonkeeper",
						name: "Tonkeeper",
						tondns: "tonkeeper.ton",
						jsBridgeKey: "tonkeeper",

						imageUrl: "https://tonkeeper.com/assets/tonconnect-icon.png",
						aboutUrl: "https://tonkeeper.com",
						universalLink: "https://app.tonkeeper.com/ton-connect",
						bridgeUrl: "https://bridge.tonapi.io/bridge",

						platforms: [
							"ios",
							"android",
							"chrome",
							"firefox",
							"safari",
							"windows",
							"macos",
							"linux",
						],
					},

					{
						appName: "tonhub",
						name: "Tonhub",
						imageUrl: "https://tonhub.com/tonconnect_logo.png",
						aboutUrl: "https://tonhub.com",
						universalLink: "https://tonhub.com/ton-connect",
						jsBridgeKey: "tonhub",
						platforms: ["ios", "android"],
						bridgeUrl: "https://connect.tonhubapi.com/tonconnect",
					},
				],
			},
		});
		//

		tonConnectUI.uiOptions = {
			// @ts-ignore
			twaReturnUrl: "https://t.me/tele_block_ai_bot",
			// twaReturnUrl: "https://tano-wallet.vercel.app/",
		};

		// Check if the wallet is connected and get the address
		// Check if the wallet is connected and get the address
		tonConnectUI.onStatusChange(async (status) => {
			// @ts-ignore
			currentConnectWalletAddress = tonConnectUI.account.address;

			// currentConnectWalletAddress="0QCnMdJCBITKDvkwhciaCi9dts3FFIPe3oJ7JPLhhdewGazE"
			console.log("current wallet address ,", currentConnectWalletAddress);

			if (currentConnectWalletAddress) {
				// Fetch the balance
				// await fetchWalletBalance(currentConnectWalletAddress);
				// await fetchGasFee();
			}
		});
	});

	async function pay() {
		try {
			// await fetchGasFee(); // Fetch the gas fee when the wallet is connected
			// const transaction = {
			// 	messages: [
			// 		{

			// 			address:
			// 				// "UQDZV_FzmyQtqssRU8EqnvAvcBOeYaH9gdEqdojdQWe9uPIE", // destination address
			// 				PUBLIC_CREDIT_ADDRESS,
			// 			amount: (0.000000001 * 1e9).toString(), //Toncoin in nanotons
			// 			payload: '',
			// 		},
			// 	],
			// };

			//-------------------------- added payload ---------------------------------
			// Define the transaction object
			let transaction;
			console.log("charge : ", charge);
			if (charge !== null) {
				let amount = (2 * 0.0001 * 1e9 * Number(charge)).toString();
				transaction = {
					messages: [
						{
							address: PUBLIC_CREDIT_ADDRESS, // Destination address
							amount: amount, // Amount in nanotons
							payload: "", // Optional payload, leave empty if not needed
							stateInit: undefined, // Optional field for contract state initialization
						},
					],
					validUntil: Date.now() + 5 * 60 * 1000, // Transaction expiration time (optional)
				};
			} 
			
			// else {
			// 	transaction = {
			// 		messages: [
			// 			{
			// 				address: PUBLIC_CREDIT_ADDRESS, // Destination address
			// 				amount: (0.0001 * 1e9).toString(), // Amount in nanotons
			// 				payload: "", // Optional payload, leave empty if not needed
			// 				stateInit: undefined, // Optional field for contract state initialization
			// 			},
			// 		],
			// 		validUntil: Date.now() + 5 * 60 * 1000, // Transaction expiration time (optional)
			// 	};
			// }

			// Optionally, you may want to check if the wallet is connected before proceeding
			if (!tonConnectUI.connected) {
				throw new Error(
					"Wallet is not connected. Please connect your wallet first.",
				);
			}

			//-------------------------------added payload ----------------------------------

			// @ts-ignore
			const result = await tonConnectUI.sendTransaction(transaction);

			console.log("transaction result : ", result);

			if (result && result.boc) {
				// Use BOC to track or decode further
				console.log("Transaction BOC:", result.boc);

				// Example of how you might send BOC to a tracker
				// const { txstatus, transactionHash } = await confirmTransaction(result.boc);

				// console.log("tx status ", transactionHash);
				// if (txstatus == "finalized"||txstatus=="still pending") {
				// const { transactionId, userId, status , msgText ,model} = req.body;
			 if (charge !== null) {
					const chargeResponse = await fetch(
						`${PUBLIC_BACKEND_BASE_URI}/update-lastused`,
						{
							method: "POST",
							headers: {
								"Content-Type": "application/json",
							},
							body: JSON.stringify({
								telegramUserName: telegramUsername,
								chatId: chatId,
							}),
						},
					);
					const response = await chargeResponse.json(); // error might occur for await
					console.log("response from update-lastused : ", response);
				}
			} else {
				console.error("Failed to retrieve transaction hash.");
			}
			// } else {
			// 	console.error("Failed to get BOC.");
			// }
			
		} catch (error) {
			console.log("error while paying tx: ", error);
		}
	}

	async function confirmTransaction(boc) {
		const fullTransaction = await fetch(
			`${PUBLIC_BACKEND_BASE_URI}/confirm-transaction`,
			{
				method: "POST",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify({
					boc: boc,
				}),
			},
		);
		const response = await fullTransaction.json();
		console.log("Transaction status:", response.status);

		if (response.status === "finalized") {
			console.log("Transaction confirmed!");
		} else {
			console.log("Transaction is still pending.");
		}
		return {
			status: fullTransaction.status,
			transactionHash: fullTransaction,
		};
	}
</script>
<head>
	<meta name="cryptomus" content="325c5b63" />
</head>
<main class="container-fluid main-content text-white">
    <!-- Header -->
    <header class="d-flex flex-wrap align-items-center py-2 px-3">
        <div class="logo">
            <!-- <img src="logo.png" alt="Logo" height="40"> -->
            <p class="display-5 logo-text">TANO</p>
        </div>
        <div class="d-flex align-items-center header-button-section">
            <button id="ton-connect" class="btn btn-primary connect-wallet-btn"></button>
        </div>
    </header>

    <!-- Main Section -->
    <section class="text-center hero-section my-5">
        <h1 class="display-4">Maximizing AI-Powered Solution,<br>Only at TANO.</h1>
        <p class="lead">Seamless, decentralized storage and AI-driven insights</p>
        <!-- Pay and Swap Buttons -->
        <div class="d-flex justify-content-center align-items-center mt-4 button-container">
             <div class="d-flex">
				<button id="pay" class="pay-button btn btn-primary pay-button mx-2" on:click={pay}>Pay Using Crypto</button>
				<!-- <button id="pay" class="pay-button btn btn-primary pay-button mx-2" on:click={pay}>Pay Using Credit Card</button> -->
			 </div>
            <button id="swap" class="swap-button btn btn-secondary swap-button mx-2"><a href="https://app.symbiosis.finance/swap">Swap</a></button>
        </div>
    </section>
</main>

<style>
/* General Styles */
/* General Styles */


/* Keep header at the top and ensure items are on the same line */
header {
    background-color: white;
    color: black;
    padding: 10px 0;
    border-radius: 10px;
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap; /* Ensure items wrap on smaller screens */
}

/* Align TANO text to the start and "Connect Wallet" button to the end */
.logo {
    flex: 1; /* Allow TANO text to take available space */
}

.header-button-section {
    display: flex;
    gap: 1rem; /* Space between buttons */
}

.connect-wallet-btn {
    background-color: #FF5A3C; /* Button color */
    color: white; /* Text color */
    border: none; /* Remove border */
    border-radius: 50px; /* Rounded corners */
    font-size: 0.8rem; /* Smaller font size */
    
    text-decoration: none; /* Remove underline */
    transition: all 0.3s ease; /* Smooth transition */
   
    font-weight: bold; /* Bold text */
}



/* Button hover effect */
.connect-wallet-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0px 5px 20px rgba(0, 0, 0, 0.2);
}

/* Background Gradient for main content */
.main-content {
    background: linear-gradient(135deg, #FF5A3C, #FF8442);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

/* Center the hero section in the remaining space below the header */
.hero-section {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: white;
    text-align: center;
    padding: 2rem;
}

/* Button styles */
.pay-button,
.swap-button {
    background-color: #FF5A3C;
    color: white;
    border-radius: 50px;
    padding: 1rem 2rem;
    font-size: 0.9rem;
    text-decoration: none;
    transition: all 0.3s ease;
	
	
}

.swap-button a {
    text-decoration: none;
    color: white;
}

.pay-button {
    background-color: black;
}

.swap-button {
    background-color: black;
}

.pay-button:hover,
.swap-button:hover {
    transform: translateY(-3px);
    box-shadow: 0px 5px 20px rgba(0, 0, 0, 0.2);
}

/* Typography */
.display-4 {
    font-size: 3rem;
    font-weight: bold;
}

.lead {
    font-size: 1.2rem;
    font-weight: 400;
}

/* Responsive */
@media (max-width: 768px) {
	.connect-wallet-btn{
		display: flex;
		flex-direction: column;
		justify-content: end;
	}
    .display-4 {
        font-size: 2rem;
    }
	.logo-text{
		font-size: small;
       
	}
    .hero-section {
        padding: 2rem 1rem;
    }
    header {
		display: flex;
        flex-direction: column; /* Stack items vertically on small screens */
        padding: 1rem; /* Adjust padding */
    }
	.logo-text{
		display: flex;
        flex-direction: column;
		align-items: start;
		font-weight: 600;
	
	}
    .header-button-section {

		display: flex;
        flex-direction: column;
        /* Make button section full width */
		align-items: start; /* Align buttons to end */
	
    }
}

</style>
