# Mocking

## Mocking modules

```ts
vi.mock('@apollo/client', async () => {
  const client = await vi.importActual('@apollo/client');
  return {
    ...client,
    useMutation: () => [vi.fn(), {}],
    useQuery: () => ({ data: {} })
  };
});

vi.mock('@gql/__generated__/schema_with_hooks', async () => {
  const schema = await vi.importActual('@gql/__generated__/schema_with_hooks');
  return {
    ...schema,
    useGetAllowedNotesForDispositionTemplateQuery: () => ({ data: {} }),
    useGetEstateChildrenCountQuery: () => ({
      data: {
        estate: { childrenCount: 5 }
      },
      loading: false
    })
  };
});

vi.mock('@features/DocumentAbstraction/hooks/getDoucment', () => ({
  useGetDocumentOrTrust: () => ({
    document: {
      kind: TrustKindEnum.REVOCABLE
    },
    loading: false
  })
}));
```

## Using generator functions and advanced setup

```tsx
const generateDistributionData = (
  createdDispositionTemplates: CreatedDispositionTemplate
): CreatedDistributionsDataType => ({
  createdDisposittionTemplates
  // preset data...
});

const setup = (
  createdDispositionTemplates: CreatedDispositionTemplate,
  distributionDetailsData?: Partial<
    NonNullable<
      NonNullable<
        ReturnType<typeof useGetDispositionTemplateDetailsQuery>['data']
      >
    >['dispositionTemplate']
  >['distributionDetails']
) => {
  vi.mocked(useGetDispositionTemplateDetailsQuery).mockReturnValue({
    data: distributionDetailsData
      ? {
          dispositionTemplate: {
            designatedFiduciaries: [],
            distributionDetails: distributionDetailsData
          }
        }
      : {}
  } as ReturnType<typeof useGetDispositionTemplateDetailsQuery>);
  return {
    user: userEvent.setup(),
    ...render(
      <DistributionDetails
        distributionData={generateDistributionData(createdDispositionTemplates)}
        onComplete={noOp} // or vi.fn()
      />
    )
  };
};
```
