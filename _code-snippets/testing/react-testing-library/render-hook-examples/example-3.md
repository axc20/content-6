# Example 3

## Test

```ts
import { renderHook } from '@testing-library/react';
import useDistributionDetailsTabs from '@features/EstateBuilder/Distributions/hooks/useDistributionDetailsTabs';
import { DispositionTemplateKindEnum } from '@gql/__generated__/schema_with_hooks';

const generateActiveDistribution = (
  kind: DispositionTemplateKindEnum
): Parameters<typeof useDistributionDetailsTabs>[0]['activeDistribution'] => ({
  id: '2',
  kind,
  parentTemplateId: '1',
  isGeneralNote: false,
  customizeIndividually: false
});

describe('useDistributionDetailsTabs', () => {
  describe('distribution target is present', () => {
    it.each<[string, DispositionTemplateKindEnum]>([
      ['Diagram Notes', DispositionTemplateKindEnum.MARITAL],
      ['Funding', DispositionTemplateKindEnum.OUTRIGHT_TO_SPOUSE],
      ['Class Distribution', DispositionTemplateKindEnum.CLASS_DISTRIBUTION],
      ['Diagram Notes', DispositionTemplateKindEnum.TO_TRUST]
    ])(
      'should set the default active tab to be %s for %s',
      (expectedTab, distributionKind) => {
        const { result } = renderHook(() =>
          useDistributionDetailsTabs({
            activeDistribution: generateActiveDistribution(distributionKind),
            initialValues: {
              targetId: '1',
              targetIsParentDocument: false
            }
          })
        );
        expect(result.current.activeTab).toBe(expectedTab);
      }
    );
  });

  describe('To Trust DT', () => {
    it('should set the default active tab to be Funding if the distribution target is not present', () => {
      const { result } = renderHook(() =>
        useDistributionDetailsTabs({
          activeDistribution: generateActiveDistribution(
            DispositionTemplateKindEnum.TO_TRUST
          ),
          initialValues: {
            targetId: undefined,
            targetIsParentDocument: undefined
          }
        })
      );
      expect(result.current.activeTab).toBe('Funding');
    });

    it('should set the default active tab to be Funding if the distribution target is a parent document', () => {
      const { result } = renderHook(() =>
        useDistributionDetailsTabs({
          activeDistribution: generateActiveDistribution(
            DispositionTemplateKindEnum.TO_TRUST
          ),
          initialValues: {
            targetId: '1'',
            targetIsParentDocument: true
          }
        })
      );
      expect(result.current.activeTab).toBe('Funding');
    });

    it('should correctly set the active tab if activeDistribution/initialValues changes', () => {
      const { result } = renderHook(() =>
        ({
          activeDistribution,
          initialValues
        }: Parameters<typeof useDistributionDetailsTabs[0]>) =>
          useDistributionDetailsTabs({
            activeDistribution,
            initialValues
          }),
        {
          initialProps: {
            activeDistribution: {
              id: '5',
              kind: DispositionTemplateKindEnum.MARITAL,
              parentTemplateId: '1',
              isGeneralNote: false,
              customizeIndividually: false
            },
            initialValues: {
              targetId: '1',
              targetIsParentDocument: false
            }
          }
        }
      );

      expect(result.current.activeTab).toBe('Diagram Notes');

      rerender({
        activeDistribution: {
          id: '3',
          kind: DispositionTemplateKindEnum.CLASS_DISTRIBUTION,
          parentTemplateId: '1',
          isGeneralNote: false,
          customizeIndividually: false
        },
        initialValues: {
          targetId: '10',
          targetIsParentDocument: false
        }
      });

      expect(result.current.activeTab).toBe('Class Distribution');
    });
  });
});
```
