# React App with Thirdweb to Interact with NFT Drop Contracts

This tutorial will guide you through setting up a React app that interacts with an NFT drop smart contract deployed using [Thirdweb](https://thirdweb.com/).

---

## Prerequisites

- [Node.js](https://nodejs.org/) installed
- Basic knowledge of React and blockchain
- A deployed NFT drop smart contract on Thirdweb

---

## Steps

### Bonus. Ethereum Sepolia Faucet

Get free Sepolia ETH sent directly to your wallet

[Web3 Faucet - Google Cloud | Ethereum Sepolia](https://cloud.google.com/application/web3/faucet/ethereum/sepolia)


### 1. Clone the base app

```bash
git clone https://github.com/thirdweb-example/vite-starter.git
cd vite-starter
```

### 2. Install Required Dependencies

```bash
yarn
```

### 3. Set Up Your NFT Drop Interaction

In `.env`, add the api key:

```javascript
VITE_TEMPLATE_CLIENT_ID="YOUR_THRIDWEB_CLIENT_ID_HERE"
```

### 4. Set Up Your NFT Drop Interaction

In `src/App.tsx`, add the following code:

```javascript
import { ConnectButton, MediaRenderer, TransactionButton, useActiveAccount, useReadContract } from "thirdweb/react";
import { sepolia } from "thirdweb/chains";
import { createWallet } from "thirdweb/wallets";

import thirdwebIcon from "./thirdweb.svg";
import { client } from "./client";
import { defineChain, getContract, toEther } from "thirdweb";
import { getContractMetadata } from "thirdweb/extensions/common"
import { claimTo, getActiveClaimCondition, getTotalClaimedSupply, nextTokenIdToMint } from "thirdweb/extensions/erc721"
import { useState } from "react";

export function App() {
	const account = useActiveAccount();

	const chain = defineChain(sepolia);
	const [quantity, setQuantity] = useState(1);

	const contract = getContract({
		client: client,
		chain: chain,
		address: "YOUR_NFT_DROP_CONTRACT_ADDRESS_HERE"
	})

	const { data: contractMetadata, isLoading: isContractMetadataLoading } = useReadContract(getContractMetadata, { contract });
	const { data: claimedSupply, isLoading: isClaimedSupplyLoading } = useReadContract(getTotalClaimedSupply, { contract });
	const { data: totalNftSupply, isLoading: isTotalSupplyLoading } = useReadContract(nextTokenIdToMint, { contract });
	const { data: claimCondition } = useReadContract(getActiveClaimCondition, { contract });

	const getPrice = (quantity: number) => {
		const total = quantity * parseInt(claimCondition?.pricePerToken?.toString() || "0")
		return toEther(BigInt(total))
	}

	return (
		<main className="p-4 pb-10 min-h-[100vh] flex items-center justify-center container max-w-screen-lg mx-auto">
			<div className="py-20">
				<Header />

				<div className="flex justify-center mb-20">
					<ConnectButton
						chain={sepolia}
						wallets={[createWallet("io.metamask"),]}
						client={client}
						appMetadata={{
							name: "Example app",
							url: "https://example.com",
						}}
					/>
				</div>
				<div className="flex flex-col items-center mt-4">
					{isContractMetadataLoading ? (
						<p>Loading...</p>
					) : (
						<>
							<MediaRenderer
								client={client}
								src={contractMetadata?.image}
								className="rounded-xl"
							/>
							<h2 className="text-2xl font-semibold mt-4">
								{contractMetadata?.name}
							</h2>
							<p className="text-lg mt-2">
								{contractMetadata?.description}
							</p>
						</>
					)}
					{isClaimedSupplyLoading || isTotalSupplyLoading ? (
						<p>Loading...</p>
					) : (
						<p className="text-lg mt-2 font-bold">
							Total NFT Supply: {claimedSupply?.toString()}/{totalNftSupply?.toString()}
						</p>
					)}
					<div className="flex flex-row items-center justify-center my-4">
						<button
							className="bg-black text-white px-4 py-2 rounded-md mr-4"
							onClick={() => setQuantity(Math.max(1, quantity - 1))}
						>-</button>
						<input
							type="number"
							value={quantity}
							onChange={(e) => setQuantity(parseInt(e.target.value))}
							className="w-10 text-center border border-gray-300 rounded-md bg-black text-white"
						/>
						<button
							className="bg-black text-white px-4 py-2 rounded-md mr-4"
							onClick={() => setQuantity(quantity + 1)}
						>+</button>
					</div>

					<TransactionButton
						transaction={() => claimTo({
						contract: contract,
						to: account?.address || "",
						quantity: BigInt(quantity),
						})}
						onTransactionConfirmed={async () => {
						alert("NFT Claimed!");
						setQuantity(1);
						}}
						onError={(e)=>{
							alert(e)
						}}
					>
						{`Claim NFT (${getPrice(quantity)} ETH)`}
					</TransactionButton>

				</div>
			</div>
		</main>
	);
}

function Header() {
	return (
		<header className="flex flex-col items-center mb-20 md:mb-20">
			<img
				src={thirdwebIcon}
				alt=""
				className="size-[150px] md:size-[150px]"
				style={{
					filter: "drop-shadow(0px 0px 24px #a726a9a8)",
				}}
			/>

			<h1 className="text-2xl md:text-6xl font-bold tracking-tighter mb-6 text-zinc-100">
				NFT Claim
				<span className="text-zinc-300 inline-block mx-1"> + </span>
				<span className="inline-block -skew-x-6 text-violet-500"> EmpeerDen </span>
			</h1>

			<p className="text-zinc-300 text-base">
				Read the{" "}
				<code className="bg-zinc-800 text-zinc-300 px-2 rounded py-1 text-sm mx-1">
					README.md
				</code>{" "}
				file to get started.
			</p>
		</header>
	);
}

```

### 5. Run the Application

```bash
npm run dev
```

Open your browser and navigate to [http://localhost:5174/](http://localhost:5174). You should now see your dapp interacting with the NFT drop contract.

---

## Notes

1. Replace `YOUR_NFT_DROP_CONTRACT_ADDRESS_HERE` with the actual address of the NFT drop contract deployed on Thirdweb.
2. Ensure you have enough funds in your wallet to cover gas fees for NFT claims.
3. Use the appropriate chain ID based on your deployment.

---

## Folder Structure

```
nft-drop-dapp
├── node_modules
├── public
├── src
│   ├── App.js
│   ├── index.css
│   ├── index.js
├── .gitignore
├── package.json
├── README.md
```

---

## Troubleshooting

- **Dependency Issues**: Ensure all dependencies are installed with `yarn`.
- **Contract Errors**: Verify the contract address and functions are correct.
- **Network Issues**: Ensure your blockchain provider is configured properly.
