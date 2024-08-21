<script>
	import { onMount } from "svelte";
	import { TonConnectUI } from "@tonconnect/ui";

	/**
	 * @type {TonConnectUI}
	 */
	let tonConnectUI;
	let currentConnectWalletAddress;
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
						bridgeUrl: "https://bridge.tonapi.io/bridge",
						platforms: ["chrome", "android"],
					},
				],
			},
		});
		currentConnectWalletAddress = tonConnectUI.account?.address;

		console.log("connected wallet address : ", currentConnectWalletAddress);
		tonConnectUI.uiOptions = {
			// @ts-ignore
			twaReturnUrl: "https://t.me/tele_block_ai_bot",
		};
	});

	async function pay() {
		const transaction = {
			messages: [
				{
					address:
						"0:412410771DA82CBA306A55FA9E0D43C9D245E38133CB58F1457DFB8D5CD8892F", // destination address
					amount: "20000000", //Toncoin in nanotons
				},
			],
		};

		const result = await tonConnectUI.sendTransaction(transaction);
	}
</script>

<main>
	<!-- Header with Connect Wallet button -->

	<div class="d-flex justify-content-end py-2" id="header">
		<button id="ton-connect" class="border-white bg-primary rounded py-1"
		></button>
	</div>

	<!-- Main content -->
	<h1 class="text-center my-5">Welcome to Tano Wallet</h1>
	<h3 class="text-center">Wallet balance: 0.00$</h3>

	<!-- Action buttons -->
	<div
		class="d-flex justify-content-center align-items-center"
		id="button-header"
	>
		<button
			class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5"
			on:click={pay}>Pay</button
		>
		<button class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5"
			>Add</button
		>
	</div>

	<!-- Footer with Close Wallet button -->
	<div class="d-flex text-center justify-content-center" id="footer">
		<button class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5"
			>Close Wallet</button
		>
	</div>
</main>
