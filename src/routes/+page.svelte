<script>
    import { onMount } from 'svelte';
    import { TonConnectUI } from '@tonconnect/ui';

    let tonConnectUI;

    onMount(() => {
        // Initialize TonConnectUI after component is mounted
       
        tonConnectUI = new TonConnectUI({
            manifestUrl: 'https://tano-wallet.vercel.app/tonconnect-manifest.json',
            buttonRootId: 'ton-connect',
            

        });
        tonConnectUI.uiOptions={
            // @ts-ignore
            twaReturnUrl: 'https://t.me/tele_block_ai_bot'
        }
    });
</script>

<main>
    <!-- Header with Connect Wallet button -->
  
    <div class="d-flex justify-content-end py-2" id="header">
        <button id="ton-connect" class="border-white bg-primary rounded py-1"></button>
    </div>

    <!-- Main content -->
    <h1 class="text-center my-5">Welcome to Tano Wallet</h1>
    <h3 class="text-center">Wallet balance: 0.00$</h3>

    <!-- Action buttons -->
    <div class="d-flex justify-content-center align-items-center" id="button-header">
        <button class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5">Pay</button>
        <button class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5">Add</button>
    </div>

    <!-- Footer with Close Wallet button -->
    <div class="d-flex text-center justify-content-center" id="footer">
        <button class="mx-5 my-5 rounded-3 bg-primary text-white p-2 px-5">Close Wallet</button>
    </div>
</main>
