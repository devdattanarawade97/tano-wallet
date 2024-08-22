<script>
	import { onMount } from "svelte";
	import { TonConnectUI } from "@tonconnect/ui";

	import { PUBLIC_CREDIT_ADDRESS } from "$env/static/public";

	/**
	 * @type {TonConnectUI}
	 */
	let tonConnectUI;
	let currentConnectWalletAddress = "";
	let chatId;
	let msgText;
	let model;
	//http://localhost:5173/?chat_id=5831161789&msg_text=hello&model=gpt
	onMount(() => {
		// Initialize TonConnectUI after component is mounted
		const urlParams = new URLSearchParams(window.location.search);
		chatId = urlParams.get("chat_id");
		msgText = urlParams.get("msg_text");
		model = urlParams.get("model");
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
						 bridgeUrl: "https://bridge.tonapi.io/bridge",
						// bridgeUrl: "https://testnet.tonapi.io/bridge",
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
						//  bridgeUrl: "https://testnet.tonapi.io/bridge",

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
				],
			},
		});

		tonConnectUI.uiOptions = {
			// @ts-ignore
			twaReturnUrl: "https://t.me/tele_block_ai_bot",
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
		// await fetchGasFee(); // Fetch the gas fee when the wallet is connected
		const transaction = {
			messages: [
				{
					address:
						// "UQDZV_FzmyQtqssRU8EqnvAvcBOeYaH9gdEqdojdQWe9uPIE", // destination address
						PUBLIC_CREDIT_ADDRESS,
					amount: (0.000000001 * 1e9).toString(), //Toncoin in nanotons
				},
			],
		};

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
			const fetchModelResponse = await fetch(
				"http://localhost:3000/notify-transaction",
				{
					method: "POST",
					headers: {
						"Content-Type": "application/json",
					},
					body: JSON.stringify({
						transactionId: "transactionHash",
						userId: chatId,
						status: "status",
						msgText: msgText,
						model: model,
					}),
				}
			);
			const response = fetchModelResponse.json();
			console.log("response from notify-transaction : ", response);
		} else {
			console.error("Failed to retrieve transaction hash.");
		}
		// } else {
		// 	console.error("Failed to get BOC.");
		// }
	}

	async function confirmTransaction(boc) {
		const fullTransaction = await fetch(
			"http://localhost:3000/confirm-transaction",
			{
				method: "POST",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify({
					boc: boc,
				}),
			}
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

<main
	class="  d-flex flex-column justify-content-between align-items-center text-white"
>
	<!-- Header with Connect Wallet button -->
	<div class="d-flex justify-content-end py-3 w-100 px-4" id="header">
		<button id="ton-connect" class="connect-button shadow-lg"></button>
	</div>

	<!-- Main content -->
	<h1 class="text-center my-2 display-4 text-glow">Welcome to Tano Wallet</h1>

	<!-- Action buttons -->
	<div
		class="d-flex justify-content-center align-items-center"
		id="button-header"
	>
		<button class="pay-button shadow-lg mx-5 my-5 fs-5 fw-bold" on:click={pay}
			>Pay</button
		>
	</div>

	<!-- Footer -->
	<footer class="text-center w-100 py-5 bg-footer-gradient">
		<h4 class="text-shadow">Powered By Tano</h4>
	</footer>
</main>

<style>
	/* Gradient background for the main container */
	.text-glow {
		color: #fff;
		text-shadow:
			0 0 10px #ff6f61,
			0 0 20px #ff6f61,
			0 0 30px #ff6f61,
			0 0 40px #ff6f61;
	}

	/* Gradient animation */
	@keyframes gradientBG {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}

	/* Glowing text effect */
	.text-glow {
		color: #fff;
		text-shadow:
			0 0 10px #ff6f61,
			0 0 20px #ff6f61,
			0 0 30px #ff6f61,
			0 0 40px #ff6f61;
	}

	/* Shading effect for the text */

	/* Button styles */
	.connect-button,
	.pay-button {
		background: linear-gradient(45deg, #ff6f61, #ff9a8b);
		border: none;
		border-radius: 50px;
		padding: 0.7rem 1.5rem;
		color: white;
		font-weight: bold;
		cursor: pointer;
		transition:
			transform 0.3s ease,
			box-shadow 0.3s ease;
	}

	.connect-button:hover,
	.pay-button:hover {
		transform: translateY(-3px);
		box-shadow: 0px 5px 20px rgba(255, 105, 135, 0.4);
	}

	/* Footer gradient */
	.bg-footer-gradient {
		background: linear-gradient(to right, #1a2a6c, #b21f1f, #fdbb2d);
	}

	/* Text shadow for footer text */
	.text-shadow {
		text-shadow: 0px 2px 4px rgba(0, 0, 0, 0.2);
	}

	/* For mobile responsiveness */
	@media (max-width: 768px) {
		.display-4 {
			font-size: 2.5rem;
		}
		.connect-button,
		.pay-button {
			padding: 0.6rem 1.2rem;
		}
	}
</style>
