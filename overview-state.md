# State Management

## 🗂️ State Management Pattern

### Zustand Stores

```typescript
// store/useFeatureStore.ts
import { create } from "zustand";

interface FeatureStore {
  data: FeatureData | null;
  setData: (data: FeatureData) => void;
}

export const useFeatureStore = create<FeatureStore>((set) => ({
  data: null,
  setData: (data) => set({ data }),
}));
```

### Rematch Models

- Untuk complex state management
- Lokasi: `rematch/models/`
