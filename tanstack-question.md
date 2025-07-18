# TanStack Query: isStale Not Updating with ForesightJS

## What I'm Building
I'm creating a component that fetches images using [ForesightJS](https://foresightjs.com/) for predictive prefetching - not on render or hover, but based on user intent prediction. The component displays all query states in the button for testing purposes.

## The Issue
All TanStack Query properties update correctly (`isFetching`, `isRefetching`, etc.) except `isStale`, which never changes from `false` even after the `staleTime` period has elapsed.

## Code Structure
```tsx
const STALE_TIME = 3000

const imageQueryOptions = (image: ForesightImage, enabled: boolean, dataUpdatedCount: number | undefined) =>
  queryOptions({
    queryKey: ["image", image],
    queryFn: async () => {
      const isEven = (dataUpdatedCount || 0) % 2 === 0
      const response = await fetch(isEven ? image.url : image.secondUrl)
      const blob = await response.blob()
      return { blob, fromUrl: isEven ? "first" : "second" }
    },
    staleTime: STALE_TIME,
    enabled: enabled,
  })

// In component:
const { data, isFetching, isStale, isRefetching } = useQuery(imageQueryOptions(image, false, 0))

// ForesightJS triggers prefetch:
useForesight({
  callback: async () => {
    await queryClient.prefetchQuery(imageQueryOptions(image, true, dataUpdateCount))
  },
  reactivateAfter: STALE_TIME
})
```

## Expected Behavior
After 3 seconds, `isStale` should become `true` to reflect that cached data is stale.

## Actual Behavior  
`isStale` remains `false` indefinitely, while other query states update normally.

Any insights on why `isStale` isn't updating in this ForesightJS + TanStack Query setup?