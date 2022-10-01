# NEFT QuickDrop + CMv2 based on Metaplex Candy Machine Reference UI (cra v4)

QuickDrop is a wallet based WL and SPL token claiming solution. This solution comes in the form of an npm package and API to allow developers freedom to create and build on top of this claiming solution. Include it in your mint page or use it in replacement of airdrops.

* Easy to use admin dashboard to create and manage your ongoing token claims. Simply prepare your list of wallets including the # of tokens each wallet gets in tsv or csv format.
* Pay the initial Sol fee to create a QuickDrop, but after that the fees get shifted over to those claiming. It is a fixed 0.## Sol fee per claim. It does not matter the quantity being claimed

***

**How to create a QuickDrop:**

1. Visit https://tokenclaim.neft.world/ and login with your Solana wallet.
2. Click on the "Create a New Drop" button and follow these steps to create a new drop:
   * Enter the Mind Id of the token you want to be claimed.
   * Enter a name for the token claim (mainly for your reference on the dashboard).
   * Click on Create and confirm the transaction for the drop creation. 
3. Upload your tsv or csv file that includes the wallets and the number of tokens each wallet should receive. An example of the csv format is below.
4. Initiate your QuickDrop. The number of tokens found in the csv or tsv file will be taken from your wallet and sent to the distributor wallet. This is the wallet that acts as escrow and distributes the token claims. Upon success, you will receive your QuickDrop ID.



```TypeScript
import { NotifyType, useTokenClaimer } from "@jeeh/tokenclaim-ui";
import { useWallet } from "@solana/wallet-adapter-react";

export const TokenClaimer = () => { 
  const{publicKey, signAllTransactions} = useWallet();
    
  const notify = (msg: string, notifyType: NotifyType) => {
        console.log({ msg, notifyType });
  };
    
  const{
       availableAmount,
       totalWhitelistedAmount,
       claimedAmount,
       onClick,
       isFetching,
       isExecuting,
        isError,
  } = useTokenClaimer(
    {
      quickdropId: "2S9wcF12tGwRnZJh16H7LVF4DCEVKvuAwfRZGaFcZ8Uq", /// replace this with your QuickDrop ID
      apiBaseUrl: "https://quickdrop.neft.world",
      solanaRpcHost: process.env.REACT_APP_SOLANA_RPC_HOST!,
    },
    notify,
    publicKey?.toBase58(),
    signAllTransactions,
    "1"
  );

  const [highlight, setHighlight] = useState<boolean>(false);
  const [lastClaimedAmount, setLastClaimedAmount] = useState<string>();

  const processHighlight = useCallback(() => {
    const isFirstLoad = lastClaimedAmount === undefined;
    setHighlight(!isFirstLoad);
    setLastClaimedAmount(claimedAmount.toString());
    return isFirstLoad;
  }, [lastClaimedAmount, claimedAmount]);

  useEffect(() => {
    let active = true;
    const isFirstLoad = processHighlight();

    if (!isFirstLoad) {
      setTimeout(() => {
        active && setHighlight(false);
      }, 350);
    }
    return () => {
      active = false;
    };
  }, [claimedAmount]);

  return (
    <Box>
      <Typography variant='h2'>Neft QuickDrop</Typography>
      <Box>
        {isError ? (
          <Box>Could not connect to quickdrop API</Box>
        ) : isFetching || isExecuting ? (
          <CircularProgress />
        ) : totalWhitelistedAmount > 0 ? (
          <Box>
            <CTAButton
              disabled={availableAmount === BigInt(0)}
              onClick={onClick}
            >
              Claim
            </CTAButton>
            <Typography variant="body1">Claimed</Typography>
            <Typography variant="h2" color={highlight ? "error" : "primary"}>
              {claimedAmount.toString()}
            </Typography>{" "}
            <Typography variant="body1">
              Remaining
              <Typography variant="h2" color={highlight ? "error" : "primary"}>
                {availableAmount.toString()}{" "}
              </Typography>
            </Typography>
          </Box>
        ) : (
          <Box>Wallet not whitelisted</Box>
        )}
      </Box>
    </Box>
  );
};
```
