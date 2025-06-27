# Render Props

The Render Props pattern is a technique for sharing code between components using a prop whose value is a function. The key advantage of this pattern is its flexibility. It allows a component to be reusable wwith different rendering logic, making it more modular and maintainable.

```jsx
const DataProvider = ({ render }) => {
  const [data, setData] = useState('data');
  // Does some logic or data fetching and passes
  // the results to the render prop function

  // the render prop is used here
  return render(data);
};

// The component that uses `DataProvider` defines what to
// render with the data it receives from `DataProvider`
// through the render prop
const App = () => {
  return (
    <div>
      <DataProvider
        render={data => (
          <div>
            <p>{data}</p>
          </div>
        )}
      />
    </div>
  );
};
```

```tsx
const FeatureFlagRender = ({
  featureFlag,
  children
}: {
  featureFlag: keyof typeof FeatureFlagNameEnum;
  children: (isEnabled: boolean) => ReactNode;
}) => {
  const featureFlags = useAllLevelFeatureFlags();
  const isEnabled = featureFlags[featureFlag];

  return <>{children(isEnabled)}</>;
};

// test
vi.mock('@hooks/useAllLevelFeatureFlags', () => ({
  default: () => ({
    [FeatureFlagNameEnum.ENABLE_AI_ASSISTED_ABSTRACTION]: false,
    [FeatureFlagNameEnum.ENABLE_TILE_REFACTOR]: true
  })
}));

describe('FeatureFlagRender', () => {
  it('should correctly conditionally render content', () => {
    render(
      <>
        <FeatureFlagRender featureFlag="ENABLE_AI_ASSISTED_ABSTRACTION">
          {isEnabled => <div>{isEnabled ? 'machines' : 'stone age'}</div>}
        </FeatureFlagRender>
        <FeatureFlagRender featureFlag="ENABLE_TILE_REFACTOR">
          {isEnabled => <div>{isEnabled ? 'enabled' : 'disabled'}</div>}
        </FeatureFlagRender>
      </>
    );
  });
  expect(screen.getByText('stone age')).toBeInTheDocument();
  expect(screen.getByText('enabled')).toBeInTheDocument();
});

// Another example:
// Distributions component internally has access to or defines values for
// deathScenario, selectedDispositionTemplateId, onClose
// and passes them to the renderEditModal prop
const Component = () => (
  <Distributions
    renderEditModal={(
      deathScenario,
      selectedDispositionTemplateId,
      onClose
    ) => (
      <LegacyEditDistributionModal
        deathScenario={deathScenario}
        selectedDispositionTemplateId={selectedDispositionTemplateId}
        onClose={onClose}
      />
    )}
  />
);
```
