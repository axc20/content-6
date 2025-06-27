# Scroll Hooks Example

## `useSyncScroll`

```ts
import { useEffect, RefObject } from 'react';
import { NOOP } from '@constants/misc';

const useSyncScroll = (
  syncFromRef: RefObject<HTMLElement>,
  syncToRef: RefObject<HTMLElement>,
  isLoading: boolean
) => {
  useEffect(() => {
    const syncFromElement = syncFromRef.current;
    const syncToElement = syncToRef.current;
    if (!syncFromElement || !syncToElement || isLoading) return NOOP;

    let isScrolling = false;

    const syncElements = (source: HTMLElement, target: HTMLElement) => {
      if (isScrolling) return;

      isScrolling = true;
      // eslint-disable-next-line no-param-reassign
      target.scrollLeft = scource.scrollLeft;
      const scrollValue = getComputedStyle(source).getPropertyValue('--scroll');
      target.style.setProperty('--scroll', scrollValue);
      isScrolling = false;
    };

    const handleFromScroll = () => syncElements(syncFromElement, syncToElement);
    const handleToScroll = () => syncElements(syncToElement, syncFromElement);

    // Initial sync
    handleFromScroll();

    return () => {
      syncFromElement.removeEventListener('scroll', handleFromScroll);
      syncToElement.removeEventListener('scroll', handleToScroll);
    };
  }, [syncFromRef, syncToRef, isLoading]);
};

export default useSyncScroll;
```

## `useScrollProperty`

```ts
import { useEffect, useRef } from 'react';
import { NOOP } from '@constants/misc';

const useScrollPropery = (isLoading: boolean) => {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const scrollContainer = ref.current;
    if (!scrollContainer || isLoading) return NOOP;

    const handleScroll = () => {
      const maxScroll =
        scrollContainer.scrollWidth - scrollContainer.clientWidth;
      const currentScroll = scrollContainer.scrollLeft;
      const scrollPercentage = currentScroll / maxScroll;

      scrollContainer.style.setProperty('--scroll', String(scrollPercentage));
    };

    const tableEl = scrollContainer.querySelector('table');
    if (tableEl) {
      const isScrollable = tableEl.clientWidth > scrollContainer.clientWidth;
      // Ensure the shadow is visible when the table is scrollable
      if (isScrollable) scrollContainer.style.setProperty('--scroll', '0.001');
    }

    scrollContainer.addEventListener('scroll', handleScroll, false);

    return () => {
      scrollContainer.removeEventListener('scroll', handleScroll, false);
    };
  }, [isLoading]);

  return {
    ref
  };
};

export default useScrollProperty;
```

## Usage

```ts
const { ref: tableScrollContainerRef } = useScrollProperty(isLoading);
const stickyHeaderRef = useRef<HTMLElement>(null);
useSyncScroll(tableScrollContainerRef, stickyHeaderRef, isLoading);
```
