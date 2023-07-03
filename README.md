# Wi-Fi

For `Decreasing Frictions in DeFi` Hackathon

# Description

# **The problem WiFi solves**
# **Problem**
DeFi has opened diverse opportunities not possible from the centralized finance systems. It is free from many regulations and products are easily customizable and flexible. However, the complicated user experience in interacting with DeFi protocols hinders DeFi’s true potential. There are three main problems in DeFi that hinders a seamless UX.

1. People who are not familiar with DeFi do not know **WHERE** to invest in. What kind of protocols there are, how they work, which one are the optimal, lucrative choice to invest in.
2. Even if they knew where to invest in, there lies the problem of **HOW** to invest. Even if they knew their source and destination in moving their asset, they would need to know how to optimally swap, bridge, aggregate assets multiple times and finally deposit assets to the DeFi protocol.
3. Finally, even if they solved the first two problems, they would need to navigate through multiple dApps, moving back and forth through different pages, which can be time consuming and stressful.

# **Solution**
Our project aggregates all the necessary components needed for a seamless DeFi experience. First by aggregating DeFi protocol information via TheGraph API and DeFi aggregator APIs, we give a diverse investment option with high APR to choose from. Next, our Funnel Algorithm aggregates source value, composed of multiple assets from multiple chains, to the destination chain. Our Funnel Contract receives assets from the Funnel Algorithm and swaps the assets to the right amount and type of asset and deposits the funds in the DeFi protocol. This whole complicated process is abstracted, and the user only needs to choose how much and where to invest in. Other utilities such as gathering multiple assets and sending it to other chains for paying gas fees and withdrawal functionality complete the seamless UX. All the functionalities mentioned are integrated into a single dApp, making the DeFi experience accessible for everyone. We were able to realize this by the help of 0x’s swap API, which gave the optimal swapping path between to different type of assets. 

# **Challenges we ran into**
# **Challenge**
1. Despite the effort to reduce the amount of transactions for a frictionless experience, the process of transferring the assets (via swap, bridging) to the DeFi protocol could not be done in a single transaction, due to bridging and transaction signatures. Swap ratio, bridging fees, gas prices are constantly changing, making these results non-deterministic which created an obstacle in depositing the exact amount of assets and optimal path in each step.
2. Testnets lacked some DeFi infrastructures we needed for our project demo.

# **Solution**
1. Through recalculation after each step, we were able to calculate the optimal path for each step and the exact amount to transfer to the next step. For example, for a path abstracted into 6 steps, initially, the optimal path for step 1-6 is calculated and step 1 is executed. After step 1, the optimal path and assets for path 2-6 are recalculated and so on until the very last step. By integrating with the 0x Swap API, we were able calculate efficiently and guarantee the optimal path.
2. We deployed and used some mock dexes and bridges.

## 0x Swap API

We used `0x Swap API` to gather several tokens into one token
Since 0x api supports few token swap pair (only ETH-USDC) on Göerli testnet, our service used 0x api only for pairs within 0x-supported tokens, and if not, we had to proceed with swapping in UniswapV2 contracts distributed by our team.

The code below is to verify that 0x api supports swap paths.
On `src/modules/taskManager/taskRouter/bundleSwapTasks.ts`,

```typescript
async function isZeroXSupported(
  chain: Chain,
  fromToken: Token,
  toToken: Token
) {
  if (!chain.zeroX) return null; // 0x unsupported chain
  const zeroXApi = axios.create({
    baseURL: chain.zeroX.apiBaseUrl,
  });

  const amountIn = fromToken.parse("0.1"); // mock amount for check 0x swap route

  try {
    const ETH = "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE";
    const sellToken =
      fromToken.address === constants.AddressZero ? ETH : fromToken.address;
    const buyToken =
      toToken.address === constants.AddressZero ? ETH : toToken.address;

    const { data: quoteData } = await zeroXApi.get<ZeroXQuoteDto>(
      `swap/v1/quote`,
      {
        params: {
          sellToken,
          buyToken,
          sellAmount: amountIn.toString(),
          skipValidation: true,
        },
      }
    );

    return quoteData;
  } catch (error) {
    return null;
  }
}
```

Once the swap path is confirmed, use the code below to run the swap.
On `src/modules/taskManager/tasks/move/ZeroXSwapTask.ts`,

```typescript
const { fromToken, toToken, amountIn, amountOut, quoteData } =
  await this.getInfo(doneAmountStatus, false);

const tx = await signer.sendTransaction({
  gasLimit: quoteData.gas,
  gasPrice: quoteData.gasPrice,
  to: quoteData.to,
  data: quoteData.data,
  value: quoteData.value,
  chainId: quoteData.chainId,
});
```

## ConnectKit

It was really easy to link connectkit to our service.  
With this hackathon, I think I will use connectkit often in the future.
On `src/App.tsx`,

```typescript
function App() {
  return (
    <WagmiConfig config={config}>
      <ConnectKitProvider theme="default" mode="dark">
        <Router />
      </ConnectKitProvider>
    </WagmiConfig>
  );
}

export default App;
```

