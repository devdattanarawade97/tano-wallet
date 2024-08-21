<script>
	import { onMount } from "svelte";
	import { TonConnectUI } from "@tonconnect/ui";

	/**
	 * @type {TonConnectUI}
	 */
	let tonConnectUI;
	let currentConnectWalletAddress = "";
	// let walletBalance;

	onMount(() => {
		// Initialize TonConnectUI after component is mounted

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

	// async function fetchWalletBalance(address) {
	// 	try {
	// 		const response = await fetch(`https://toncenter.com/api/v2/getAddressBalance?address=${address}`);
	// 		const data = await response.json();

	// 		const balanceNano = data.result;
	// 		walletBalance = (parseInt(balanceNano, 10) / 1e9).toFixed(2) + " TON";

	// 	} catch (error) {
	// 		console.error("Error fetching wallet balance:", error);
	// 		walletBalance = "Error fetching balance";
	// 	}
	// }
	async function pay() {
		// await fetchGasFee(); // Fetch the gas fee when the wallet is connected
		const transaction = {
			messages: [
				{
					address:
						// "UQDZV_FzmyQtqssRU8EqnvAvcBOeYaH9gdEqdojdQWe9uPIE", // destination address
						"0:cffd1b65b4206481aa807dea621642ee6bbd3ff2f18deef19b696d0c0624b37f",
					amount: (0.001 * 1e9).toString(), //Toncoin in nanotons
				},
			],
		};

		const result = await tonConnectUI.sendTransaction(transaction);

		console.log("transaction result : ", result);
	}

	// async function fetchGasFee() {
	// 	try {
	//         const response = await fetch('https://toncenter.com/api/v2/estimateFee', {
	//         method: 'POST',
	//         headers: {
	//             'accept': 'application/json',
	//             'Content-Type': 'application/json'
	//         },
	//         body: JSON.stringify({
	//             address: currentConnectWalletAddress,
	//             body: 'te6ccgEBAQEAAgAAAA==', // Example minimal body encoded in base64
	//             init_code: '', // If there's initial code, provide it
	//             init_data: '', // If there's initial data, provide it
	//             ignore_chksig: true // Assuming you want to ignore the signature check
	//         })
	//     });

	//     const data = await response.json();
	//     console.log('gas data result ,',data);
	//     // Assuming the fee is returned in nanotons
	//     const feeNano = data.result?.fee || 0;
	//     gasFee = (parseInt(feeNano, 10) / 1e9).toFixed(2) + " TON";

	// 	} catch (error) {
	// 		console.error("Error fetching gas fee:", error);
	// 		gasFee = "Error fetching gas fee";
	// 	}
	// }
</script>

<main class="bg-black text-white vh-100">
	<!-- Header with Connect Wallet button -->

	<div class="d-flex justify-content-end py-2 " id="header">
		<button id="ton-connect" class="border-white bg-primary rounded py-1"
		></button>
	</div>

	<!-- Main content -->
	<h1 class="text-center my-5">Welcome to Tano Wallet</h1>
	<!-- <h3 class="text-center">Estimated Gas Fee: {gasFee}</h3> -->

	<!-- Action buttons -->
	<div
		class="d-flex justify-content-center align-items-center"
		id="button-header"
	>
		<button
			class="mx-5 my-5 rounded-5 fs-6 fw-bold bg-primary text-white py-2 px-5 w-auto border-white"
			on:click={pay}>Pay</button
		>
        
	</div>
  
    <footer class="text-center">
        <h4>Powered By Tano</h4>
    </footer>
</main>
