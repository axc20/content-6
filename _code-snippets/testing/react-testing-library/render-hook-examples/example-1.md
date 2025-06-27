# Example 1

## Hook

```ts
const useCategorizedAllowedNotes = ({
  dispositionTemplate,
  gstFlags
}: {
  dispositionTemplate: SelectedDispositionTemplateForNotes;
  gstFlags: string[];
}) => {
  const { id, isGeneralNote, separateTrustId } =
    getNotesGqlVariables(dispositionTemplate);
  const { data } = useGetAllowedNotesForDispositionTemplateQuery({
    variables: {
      id,
      isGeneralNote,
      separateTrustId
    },
    fetchPolicy: 'cache-and-network',
    skip: !dispositionTemplate.id
  });

  const { notes, notesCategories } = useMemo(() => {
    if (!data?.dispositionTemplate?.allowedOrganizationProvisions) {
      return {
        notes: [],
        notesCategories: []
      }
    }

    return {
      notes: data.dispositionTemplate.allowedOrganizationProvisions,
      notesCategories: [
        'General',
        ...new Set(
          data.dispositionTemplate.allowedOrganizationProvisions
            .map(note => note.provisionCategory?.text)
            .filter((category): category is string => category !== undefined)
        )
      ]
    }
  }), [data?.dispositionTemplate?.allowedOrganizationProvisions];

  const { hasNotesToDisplay, notesByCategory, customNotes } = useMemo(() => {
    const displayableNotes = notes.filter(note =>
      gstFlags.includes(note.gstFlag)
    );
    const hasNotesToDisplay = displayableNotes.length > 0;
    const notesByCategory = notesCategories.reduce(
      (acc, category) => ({
        ...acc,
        [category]: []
      }),
      {} as Record<string, AllowedOrganizationProvisions>
    )

    displayableNotes.forEach(note => {
      if (note.fromCustomId) {
        customNotes.push(note);
        return;
      }

      const category = note.provisionCategory?.text || 'General';

      notesByCategory[category].push(note);
    });

    return { hasNotesToDisplay, notesByCategory, customNotes }
  }, [gstFlags, notes, notesCategories]);

  return {
    hasNotesToDisplay,
    notesByCategory,
    customNotes
  }
};

export default useCategorizedAllowedNotes;
```

## Test

```ts
vi.mock('@gql/__generated__/schema_with_hooks', async () => {
  const schema = await vi.importActual('@gql/__generated__/schema_with_hooks');
  return {
    ...schema,
    useGetAllowedNotesForDispositionTemplateQuery: vi.fn()
  };
});

const testProvisionCategory: AllowedOrganizationProvisions[0]['provisionCategory'] =
  {
    id: '1',
    text: 'Test Category',
    __typename: 'ProvisionCategory'
  };

const generateDefaultProvisionNote = ({
  id,
  isCustom,
  gstFlag,
  withCategory
}: {
  id: string;
  isCustom?: boolean;
  gstFlag?: OrganizationProvisionGstFlagEnum;
  withCategory?: boolean;
}): AllowedOrganizationProvisions[0] => ({
  id,
  text: withCategory
    ? `${testProvisionCategory.text} Note ${id}`
    : `${isCustom ? 'Custom' : 'General'} Note ${id}`,
  gstFlag: gstFlag ?? OrganizationProvisionGstFlagEnum.GENERAL,
  fromCustomId: isCustom ? '99' : undefined,
  provisionCategory: withCategory ? testProvisionCategory : null,
  __typename: 'OrganizationProvision'
});

const setup = ({
  allowedOrganizationProvisions,
  gstFlags
}: {
  allowedOrganizationProvisions: AllowedOrganizationProvisions;
  gstFlags?: string[];
}) => {
  vi.mocked(useGetAllowedNotesForDispositionTemplateQuery).mockReturnValue({
    data: { dispositionTemplate: { allowedOrganizationProvisions } }
  } as unknown as ReturnType<typeof useGetAllowedNotesForDispositionTemplateQuery>);

  return renderHook(() =>
    useCategorizedAllowedNotes({
      dispositionTemplate: {
        id: '2',
        isGeneralNote: false,
        parentTemplateId: '1'
      },
      gstFlags: gstFlags || [OrganizationProvisionGstFlagEnum.GENERAL]
    })
  ;)
};

describe('useCategorizedAllowedNotes', () => {
  it('should return hasNotesToDisplay as false if there are no allowedOrganizationProvisions', () => {
    const { result } = setup({ allowedOrganizationProvisions: [] });

    expect(result.current).toEqual({
      hasNotesToDisplay: false,
      notesByCategory: {
        General: []
      },
      customNotes: []
    });
  });

  it('should return categorized notes', () => {
    const testCategoryNote1 = generateDefaultProvisionNote({
      id: '5',
      withCategory: true
    });
    const testCategoryNote2 = generateDefaultProvisionNote({
      id: '6',
      withCategory: true
    });
    const generalNote = generateDefaultProvisionNote({ id: '1' });
    const customNote1 = generateDefaultProvisionNote({
      id: '15',
      isCustom: true
    });
    const customNote2 = generateDefaultProvisionNote({
      id: '16',
      isCustom: true
    });

    const { result } = setup({
      allowedOrganizationProvisions: [
        testCategoryNote1,
        testCategoryNote2,
        generalNote,
        customNote1,
        customNote2
      ]
    });

    expect(result.current).toEqual({
      hasNotesToDisplay: true,
      notesByCategory: {
        General: [generalNote],
        [testProvisionCategory.text]: [testCategoryNote1, testCategoryNote2]
      },
      customNotes: [customNote1, customNote2]
    })
  });

  it('should return categorized notes filtered by gstFlag', () => {
    const testCategoryGeneralNote = generateDefaultProvisionNote({
      id: '5',
      withCategory: true
    });
    const testCategoryGstNote = generateDefaultProvisionNote({
      id: '6',
      gstFlag: OrganizationProvisionGstFlagEnum.GST,
      withCategory: true
    });
    const testCategoryNonGstNote = generateDefaultProvisionNote({
      id: '7',
      gstFlag: OrganizationProvisionGstFlagEnum.NON_GST,
      withCategory: true
    });
    const generalGeneralNote = generateDefaultProvisionNote({ id: '1' });
    const generalGstNote = generateDefaultProvisionNote({
      id: '2',
      gstFlag: OrganizationProvisionGstFlagEnum.GST
    });
    const generalNonGstNote = generateDefaultProvisionNote({
      id: '3',
      gstFlag: OrganizationProvisionGstFlagEnum.NON_GST
    });
    const customGeneralNote = generateDefaultProvisionNote({
      id: '15',
      isCustom: true
    });
    const customGstNote = generateDefaultProvisionNote({
      id: '16',
      gstFlag: OrganizationProvisionGstFlagEnum.GST,
      isCustom: true
    });
    const customNonGstNote = generateDefaultProvisionNote({
      id: '17',
      gstFlag: OrganizationProvisionGstFlagEnum.NON_GST,
      isCustom: true
    });

    const { result: result1 } = setup({
      allowedOrganizationProvisions: [
        testCategoryGeneralNote,
        testCategoryGstNote,
        testCategoryNonGstNote,
        generalGeneralNote,
        generalGstNote,
        generalNonGstNote,
        customGeneralNote,
        customGstNote,
        customNonGstNote
      ]
    });

    expect(result1.current).toEqual({
      hasNotesToDisplay: true,
      notesByCategory: {
        General: [generalGeneralNote],
        [testProvisionCategory.text]: [testCategoryGeneralNote]
      },
      customNotes: [customGeneralNote]
    });

    const { result: result2 } = setup({
      allowedOrganizationProvisions: [
        testCategoryGeneralNote,
        testCategoryGstNote,
        testCategoryNonGstNote,
        generalGeneralNote,
        generalGstNote,
        generalNonGstNote,
        customGeneralNote,
        customGstNote,
        customNonGstNote
      ],
      gstFlags: [OrganizationProvisionGstFlagEnum.GST, OrganizationProvisionGstFlagEnum.NON_GST]
    });

    expect(result2.current).toEqual({
      hasNotesToDisplay: true,
      notesByCategory: {
        General: [generalGstNote, generalNonGstNote],
        [testProvisionCategory.text]: [testCategoryGstNote, testCategoryNonGstNote]
      },
      customNotes: [customGstNote, customNonGstNote]
    });
  });
});
```
