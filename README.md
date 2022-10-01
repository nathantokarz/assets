# NEFT QuickDrop + CMv2 based on Metaplex Candy Machine Reference UI (cra v4)

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
