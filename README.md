# Parallel

A learning note of R "parallel" package.

## Initiate Cluster

makeCluster(ncores, type ="FORK")

If you are running R on Linux platform, "FORK" is available. Under this mode, imported environment and data are automatically shared by all threads. 

* However, if your program requires to read large data, and you have already read in all your data before you initiating your cluster, your cluster may be overload. Each thread will have a copy of your data, which means extensive memory usage.

* Another problem is that if you are doing simple calculation, which means the time taken by each round is very short, most of your time will be wasted on returning data from each sub-processes to the main-process.

Therefore, you will find your program time and memory consuming.

To solve this problem, a strategy is to read in your data after making cluster and not to share them across threads (usually you don't need the whole data when doing simple calculation, but it is not the case for complicated jobs). For example, 
 
 Assuming we have a large matrix and we need to do calculation by row.
 1. A bad case
 
          largeMatrix <- readRDS("largeMatrix.rds")
          cl <- makeCluster(ncores, type ="FORK")
          rowResults <- clusterApply(cl, largeMatrix, 1, calculationFunc)
          stopCluster(cl)

 2. A good case
          
          splitCal <- function(core, largeMatrix, totalR, meanR){
            start <- (core-1)*meanR+1
            end <- min(core*meanR, totalR)
            smallMatrix <- readRDS("largeMatrix.rds")[start:end,]
            splitResults <- apply(smallMatrix, 1, calculationFunc)
            return(splitResults)
          }
          
          totalR <- nrow(readRDS("largeMatrix.rds"))
          meanR <- round(totalR/ncores, 0) + 1
          
          cl <- makeCluster(ncores, type ="FORK")
          rowResults <- clusterMap(cl, splitCal, 1：ncores, 
                                   MoreArgs = list(largeMatrix = "largeMatrix.rds",
                                                   totalR = totalR, meanR = meanR),
                                   SIMPLIFY = T, USE.NAMES = F)
          stopCluster(cl)
