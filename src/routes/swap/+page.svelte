<script>
    import { onMount } from "svelte";
    import { Buffer } from 'buffer';
    import { Symbiosis } from "symbiosis-js-sdk";
  
   
  
    let symbiosis;
    let isConnected = false;
    let amount = "";
    let fromToken = "TON";
    let toToken = "ETH";
    let walletAddress = "";
  
    async function initSymbiosis() {
      symbiosis = new Symbiosis("mainnet", "my-ton-dex-app");
    }
  
    async function connectWallet() {
      try {
        const wallet = await symbiosis.connectWallet();
        walletAddress = wallet.address;
        isConnected = true;
      } catch (error) {
        console.error("Error connecting to wallet:", error);
      }
    }
  
    async function executeTrade() {
      try {
        if (!isConnected) {
          await connectWallet();
        }
  
        const tradeParams = {
          fromToken,
          toToken,
          amount: amount * 1e9,
        };
  
        const tradeResult = await symbiosis.trade(tradeParams);
        console.log("Trade successful:", tradeResult);
      } catch (error) {
        console.error("Error executing trade:", error);
      }
    }
  
    onMount(() => {
        window.Buffer = Buffer;
       initSymbiosis();
    });
  </script>
  
  <style>
    button {
      padding: 10px 20px;
      background-color: #0088cc;
      color: white;
      border: none;
      cursor: pointer;
      border-radius: 5px;
      margin-top: 10px;
    }
  
    button:hover {
      background-color: #005f99;
    }
  
    input, select {
      padding: 10px;
      margin-top: 10px;
      width: 100%;
      max-width: 300px;
      box-sizing: border-box;
    }
  
    label {
      margin-top: 15px;
      font-weight: bold;
    }
  
    .container {
      max-width: 400px;
      margin: auto;
      padding: 20px;
      background-color: #f7f7f7;
      border-radius: 8px;
    }
  
    .wallet-info {
      margin-top: 10px;
      font-weight: bold;
      color: green;
    }
  </style>
  
  <div class="container">
    {#if isConnected}
      <div class="wallet-info">Connected Wallet: {walletAddress}</div>
    {/if}
    <button on:click={connectWallet}>
      {isConnected ? 'Reconnect Wallet' : 'Connect Wallet'}
    </button>
  
    <label for="fromToken">From Token:</label>
    <select id="fromToken" bind:value={fromToken}>
      <option value="TON">TON</option>
      <option value="ETH">ETH</option>
    </select>
  
    <label for="toToken">To Token:</label>
    <select id="toToken" bind:value={toToken}>
      <option value="ETH">ETH</option>
      <option value="TON">TON</option>
    </select>
  
    <label for="amount">Amount:</label>
    <input
      type="number"
      id="amount"
      bind:value={amount}
      placeholder="Enter amount"
    />
  
    <button on:click={executeTrade} disabled={!amount || !isConnected}>
      Trade on TON DEX
    </button>
  </div>
  