# Example 2

## Hook

```ts
const useCheckedNotesIds = ({
  initialOrderedIds,
  isOpen
}: {
  initialOrderedIds: string[];
  isOpen: boolean;
}) => {
  const [notesState, setNotesState] = useState<{
    checkedIds: string[];
    deselectedIds: string[];
  }>({
    checkedIds: [],
    deselectedIds: []
  });

  useEffect(() => {
    setNotesState(prev => {
      if (isOpen) {
        // prev notes state contains all locally checked ids
        // initialOrderedIds contains actual checked ids from query, which might not
        // reflect the current checked ids in local state
        // However, initialOrderedIds may contain invalid ids that were applied from
        // another trust and need to be preserved
        const initialInvalidIds = initialOrderedIds.filter(
          id => ![...prev.checkedIds, ...prev.deselectedIds].includes(id)
        );

        return {
          ...prev,
          checkedIds: [...initialInvalidIds, ...prev.checkedIds]
        };
      }

      return {
        checkedIds: [],
        deselectedIds: []
      };
    });
  }, [initialOrderedIds, isOpen]);

  const handleNoteToggle = (id: string) => {
    setNotesState(prev => {
      if (prev.checkedIds.includes(id)) {
        return {
          checkedIds: prev.checkedIds.filter(noteId -> noteId !== id),
          deselectedIds: [...prev.deselectedIds, id]
        }
      }

      return {
        checkedIds: [...prev.checkedIds, id],
        deselectedIds: prev.deselectedIds.filter(noteId => noteId !== id)
      }
    });
  };

  const handleNotesMultiEdit = (
    createdNoteIds: string[],
    deletedNoteIds: string[]
  ) => {
    setNotesState(prev => ({
      ...prev,
      checkedIds: [
        ...prev.checkedIds.filter(id => !deletedNoteIds.includes(id)),
        ...createdNoteIds
      ]
    }));
  };

  const handleNotesApplication = (noteIds: string[]) => {
    setNotesState(prev => ({
      ...prev,
      checkedIds: [
        ...prev.checkedIds.filter(id => !noteIds.includes(id)),
        ...noteIds
      ]
    }));
  };

  return {
    checkedNotesIds: notesState.checkedIds,
    setNotesState,
    handleNoteToggle,
    handleNotesMultiEdit,
    handleNotesApplication
  };
};

export default useCheckedNotesIds;
```

## Test

```ts
const initialOrderedIds = ['3', '30', '5', '99'];

const setup = () =>
  renderHook(() =>
    useCheckedNotesIds({
      initialOrderedIds,
      isOpen: true
    })
  );

describe('useCheckedNotesIds', () => {
  it('should initialize checkedNotesIds to the initialOrderedIds when isOpen is true', () => {
    const { result } = setup();

    expect(result.current.checkedNotesIds).toEqual(initialOrderedIds);
  });

  it('should reset checkedNotesIds to an empty array when isOpen is false', () => {
    const { result } = renderHook(() =>
      useCheckedNotesIds({
        initialOrderedIds,
        isOpen: false
      })
    );

    expect(result.current.checkedNotesIds).toEqual([]);
  });

  it('handleNoteToggle should correctly update checkedNotesIds', async () => {
    const { result } = setup();
    result.current.handleNoteToggle('3');

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual(['30', '5', '99']);
    });

    result.current.handleNoteToggle('3');

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual(['30', '5']);
    });

    // re-checking the notes adds them to the back of the list in the order they were checked
    result.current.handleNoteToggle('3');
    result.current.handleNoteToggle('99');

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual(['30', '5', '3', '99']);
    });
  });

  it('handleNotesMultiEdit should correctly update checkedNotesIds - ids are added/deleted accordingly', async () => {
    const { result } = setup();
    // create notes with ids 100, 101, 102, and delete notes with ids 30, 99
    result.current.handleNotesMultiEdit(['100', '101', '102'], ['30', '99']);

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual([
        '3',
        '5',
        '100',
        '101',
        '102'
      ]);
    });
  });

  it('handleNotesApplication should correctly update checkedNotesIds - ids are added to the back of the list with order preserved', async () => {
    const { result } = setup();
    result.current.handleNotesApplication(['1', '2', '30', '3', '4']);

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual([
        '5',
        '99',
        '1',
        '2',
        '30',
        '3',
        '4'
      ]);
    });
  });

  it('should be able to handle multiple combined operations', async () => {
    const { result, rerender } = setup();
    result.current.handleNoteToggle('3');
    result.current.handleNotesMultiEdit(['100', '101'], ['99']);

    // rerender to simulate reinitialization of initialOrderedIds (since '99' was deleted)
    rerender({
      initialOrderedIds: ['3', '30', '5'],
      isOpen: true
    });

    await waitFor(() => {
      // checkedNotesIds still tracking '3' as deselected, even though it is still in initialOrderedIds
      expect(result.current.checkedNotesIds).toEqual(['30', '5', '100', '101']);
    });

    result.current.handleNotesApplication(['1', '2', '30', '3']);

    rerender({
      initialOrderedIds: ['3', '30', '5'],
      isOpen: true
    });

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual([
        '5',
        '100',
        '101',
        '1',
        '2',
        '30',
        '3'
      ]);
    });

    result.current.handleNoteToggle('1');
    result.current.handleNoteToggle('30');
    result.current.handleNoteToggle('100');
    result.current.handleNoteToggle('100');
    result.current.handleNoteToggle('30');

    await waitFor(() => {
      expect(result.current.checkedNotesIds).toEqual([
        '5',
        '101',
        '2',
        '3',
        '100',
        '30'
      ]);
    });
  });
});
```
