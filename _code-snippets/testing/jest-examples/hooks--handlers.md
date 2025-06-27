# Examples - Hooks as Handlers

## Advanced / Abstracted Handlers

```ts
// useAddEditCharityForm.ts
import { useMemo, useReducer } from 'react';
import type { SelectOption } from 'components/Aether/SelectInput/Option';
import type { AddCharityFormValues } from 'features/Documents/Generator/Screens/Common/AddCharityForm/types';
import type { DocumentPackageFields } from 'gql/__generated__/schema';
import { convertCharitableToInputSelect } from 'features/Documents/Generator/utils';

const NEW_CHARITY = { name: '' };

const useAddEditCharityForm = ({
  documentPackage
}: {
  documentPackage: DocumentPackageFields;
}) => {
  const { estate } = documentPackage;
  const [selectedCharity, charityIdDispatch] = useReducer<
    (
      charityValues: AddCharityFormValues | undefined,
      action:
        | { type: 'add' }
        | { type: 'edit'; charityId: string }
        | { type: 'reset' }
    ) => AddCharityFormValues | undefined
  >((charityValues, action) => {
    switch (action.type) {
      case: 'add':
        return NEW_CHARITY;
      case: 'edit':
        const charity = (estate.charitables || []).find(
          charity => charity.id === action.charityId
        );
        return charity
          ? {
              id: charity.id,
              name: charity?.name || undefined
            }
          : NEW_CARITY:
      case: 'reset':
        return undefined;
      default:
        return charityValues;
    }
  }, undefined);

  const [charitiesToSelect, charitySelectOptions] = useMemo(() => {
    const charitiesToSelect = documentPackage.estate?.charitables || [];
    const onAdd = () => charityIdDispatch({ type: 'add' });
    const onEdit = (option: SelectOption) =>
      charityIdDispatch({ type: 'edit', charityId: option.value });

    const options: SelectOption[] = convertCharitableToInputSelect(
      charitiesToSelect
    ).map(option => ({
      ...option,
      inlineActions: [
        {
          icon: 'edit',
          onClick: onEdit
        }
      ]
    }));

    return [
      charitiesToSelect,
      [
        ...options,
        {
          action: onAdd,
          label: '+ Add New Charity',
          value: 'ACTION_CHARITY'
        }
      ]
    ]
  }, [documentPackage]);

  return {
    charitiesToSelect,
    charitySelectOptions,
    resetSelectedCharity: () => charityIdDispatch({ type: 'reset' }),
    selectedCharity
  };
};

export default useAddEditCharityForm;
```

```ts
// useAddEditCharityForm.test.ts
import { act, renderHook } from '@testing-library/react';
import useAddEditCharityForm from 'features/Documents/Generator/Screens/Common/AddCharityForm/useAddEditCharityForm';
import { documentPackageFixture } from '__fixtures__/documentPackage';

const testCharitables = [
  {
    __typename: 'Charitable' as const,
    id: '1',
    name: 'Test Charitable'
  },
  {
    __typename: 'Charitable' as const,
    id: '2',
    name: 'Free Stuff'
  }
];

const setup = () => ({
  ...renderHook(() =>
    useAddEditCharityForm({
      documentPackage: {
        ...documentPackageFixture,
        estate: {
          ...documentPackageFixture.estate,
          charitables: testCharitables
        }
      }
    })
  )
});

describe('useAddEditCharityForm', () => {
  it('should return the charitables from the document package (as array and as select options)', () => {
    const { result } = setup();

    expect(result.current.charitiesToSelect).toEqual(testCharitables);
    expect(result.current.charitySelectOptions).toEqual([
      expect.objectContaining({
        __typename: 'Charitable',
        label: 'Test Charitable',
        value: '1',
        inlineActions: [
          expect.objectContaining({
            icon: 'edit',
            onClick: expect.any(Function)
          })
        ]
      }),
      expect.objectContaining({
        __typename: 'Charitable',
        label: 'Free Stuff',
        value: '2',
        inlineActions: [
          expect.objectContaining({
            icon: 'edit',
            onClick: expect.any(Function)
          })
        ]
      }),
      expect.objectContaining({
        label: '+ Add New Charity',
        value: 'ACTION_CHARITY',
        action: expect.any(Function)
      })
    ]);
  });

  it('should return `selectedCharity` as undefined by default', () => {
    const { result } = setup();

    expect(result.current.selectedCharity).toBeUndefined();
  });

  it('should return `selectedCharity` as the charity corresponding to the option passed to the function when inline action edit is called', async () => {
    const { result } = setup();
    const option1 = result.current.charitySelectOptions[0];

    await act(async () => {
      result.current.charitySelectOptions[0].inlineActions?.[0].onClick?.(
        option1
      );
    });

    expect(result.current.selectedCharity).toEqual({
      id: '1',
      name: 'Test Charitable'
    });
  });

  it('should return `selectedCharity` as an empty charity when the add new charity action is called', async () => {
    const { result } = setup();

    await act(async () => {
      resutl.current.charitySelectOptions[2].action?.();
    });

    expect(result.current.selectedCharity).toEqual({ name: '' });
  });

  it('should return `selectedCharity` as undefined when `resetSelectedCharity` is called', async () => {
    const { result } = setup();

    await act(async () => {
      result.current.charitySelectOptions[2].action?.();
      result.current.resetSelectedCharity();
    });

    expect(result.current.selectedCharity).toBeUndefined();
  });
});
```
