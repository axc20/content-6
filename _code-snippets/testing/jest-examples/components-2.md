# Testing Components - Example 2

```tsx
import { useQuery } from '@apollo/client';
import { MockedProvider, MockedResponse } from '@apollo/client/testing';
import {
  render,
  screen,
  waitFor,
  waitForElementToBeRemoved,
  within
} from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import PrimarySecondaryPersonScreen from 'features/Documents/Generator/Screens/PrimarySecondaryPerson';

// Note: mock functions should be prefixed with "mock"
const mockSubmitNextQuestion = jest.fn();
const mockSubmitPrevQuestion = jest.fn();

const PrimarySecondaryPersonScreenWrapper = ({
  answer
}: {
  answer: AnswerType;
}) => {
  const { data, loading } = useQuery<
    GetDocumentPackage,
    GetDocumentPackageVariables
  >(GET_DOCUMENT_PACKAGE, { variables: { id: '1' } });
  if (!data?.documentPackage || loading) return null;
  return (
    <PrimarySecondaryPersonScreen
      answer={answer}
      documentPackage={data.documentPackage}
      question={primarySecondaryPersonQuestion}
      submitNextQuestion={mockSubmitNextQuestion}
      submitPrevQuestion={mockSubmitPrevQuestion}
    />
  );
};

const setup = ({
  answer,
  additionalMocks
}: {
  answer?: AnswerType;
  additionalMocks?: MockedResponse[];
}) => ({
  user: userEvent.setup(),
  ...render(
    <MockedProvider mocks={[mockGetDocumentPackage(), ...(additionalMocks || [])]}>
      <PrimarySecondaryPersonScreenWrapper answer={answer} />
    </MockedProvider>
  );
});

beforeAll(() => {});
afterAll(() => {});

describe('PrimarySecondaryPersonScreen', () => {
  it('should render correctly', async () => {
    setup({ answer: [] });
    const question = primarySecondaryPersonQuestion.questionText;
    const tableEmptyStateText = /No fiduciaries to show yet./i;

    await waitFor(async () => {
      expect(screen.getByText(question)).toBeInTheDocument();
      expect(screen.getByText(/^Fiduciary$/i)).toBeInTheDocument();
      expect(screen.getByText(tableEmptyStateText)).toBeInTheDocument();
      expect(screen.getByText(/\+ Add Fiduciary/i)).toBeInTheDocument();
    });
  });

  it('should prepopulate the table with the answer', async () => {
    setup({ answer: primarySecondaryPersonAnswer });

    await waitFor(async () => {
      expect(screen.getByText(/Mary Smith/)).toBeInTheDocument();
      expect(screen.getByText(/Primary Fiduciary/)).toBeInTheDocument();
    });
  });

  it('should allow the user to add, edit, and remove fiduciaries', async () => {
    const { user } = setup({
      answer: primarySecondaryPersonAnsweer,
      additionalMocks: [
        mockUpdatePersonCodegenV1({
          id: BOB_SMITH.id,
          attributes: {
            address1: undefined,
            address2: undefined,
            name: BOB_SMITH.name,
            phoneNumber: undefined,
            state: undefined,
            zip: undefined
          }
        })
      ]
    });

    await screen.findByText(/Mary Smith/);
    await user.click(screen.getByText(/\+ Add Fiduciary/i));

    // 1st modal
    expect(screen.getByText(fidModalOneDescription)).toBeInTheDocument();
    expect(screen.getByText(/Select Person/)).toBeInTheDocument();

    await user.click(screen.getByRole('combobox')); // Select Person input
    await user.click(screen.getByText(/Bob Smith/));
    await user.click(screen.getByText(/Select$/));

    // 2nd modal
    expect(screen.getByText(fidModalTwoDescription)).toBeInTheDocument();

    await user.click(screen.getAllByRole('combobox')[0]); // Select Role input
    await user.click(screen.getByText(/^Backup Fiduciary$/));
    await user.click(screen.getByText(/Save/));
    await waitForElementToBeRemoved(() => screen.queryByText(fidModalTwoDescription));

    expect(screen.getByText(/Mary Smith/)).toBeInTheDocument();
    expect(screen.getByText(/Primary Fiduciary/)).toBeInTheDocument();
    expect(screen.getByText(/Bob Smith/)).toBeInTheDocument();
    expect(screen.getByText(/Backup Fiduciary/)).toBeInTheDocument();

    const editIconButtons = screen
      .getAllByRole('button')
      .filter(button => within(button).queryByText(/edit$/));
    await user.click(editIconButtons[0]); // Edit Mary Smith
    await user.click(screen.getAllByRole('combobox')[0]);
    await user.click(screen.getByText(/Secondary Backup Fiduciary/));
    await user.click(screen.getByText(/Save/));
    await waitForElementToBeRemoved(() => screen.queryByText(fidModalTwoDescription));

    expect(screen.getByText(/Mary Smith/)).toBeInTheDocument();
    expect(screen.getByText(/Secondary Backup Fiduciary/)).toBeInTheDocument();
    expect(screen.getByText(/Bob Smith/)).toBeInTheDocument();
    expect(screen.getByText(/Backup Fiduciary/)).toBeInTheDocument();

    const deleteIconButtons = screen
      .getAllByRole('button')
      .filter(button => within(button).queryByText(/delete$/));
    await user.click(deleteIconButtons[0]); // Remove Bob Smith (fids are sorted by role)

    expect(screen.getByText(/Mary Smith/)).toBeInTheDocument();
    expect(screen.getByText(/Secondary Backup Fiduciary/)).toBeInTheDocument();
    expect(screen.queryByText(/Bob Smith/)).not.toBeInTheDocument();
    expect(screen.queryByText(/Backup Fiduciary/)).not.toBeInTheDocument();

    await user.click(screen.getByText(/Continue/));
    await waitFor(() => {
      expect(screen.getByText(fidErrorMsg)).toBeInTheDocument();
    });

    await user.click(screen.getByText(/edit$/)); // Edit Mary Smith
    await user.click(screen.getByRole('combobox')[0]);
    await user.click(screen.getByText(/^Primary Fiduciary$/));
    await user.click(screen.getByText(/Save/));
    await waitForElementToBeRemoved(() => screen.queryByText(fidModalTwoDescription));
    await user.click(screen.getByText(/Continue/));

    const parsedArg = JSON.parse(mockSubmitNextQuestion.mock.calls[0][0]);

    expect(mockSubmitNextQuestion).toHaveBeenCalledTimes(1);
    expect(parsedArg[0].id).toBe(MARY_SMITH.id);
    expect(parsedArg[0].role).toBe('Primary');
  });

  it('should allow the user to add a new person as a fiduciary', async () => {
    const { user } = setup({
      answer: [],
      additionalMocks: [
        // ...
      ]
    });

    await screen.findByText(/\+ Add Fiduciary/i);
    await user.click(screen.getByText(/\+ Add Fiduciary/i));
    await user.click(screen.getByRole('combobox'));
    await user.click(screen.getByText(/\+ Add New Person/));
    await user.type(screen.getByLabelText(/Full Legal Name \*/), 'Test Person');
    await user.type(
      screen.getByPlaceholderText(/MM\/DD\/YYYY/),
      '{arrowLeft}{arrowLeft}07011982'
    );
    await user.click(screen.getByText(/Save/));
    await waitForElementToBeRemoved(() => screen.queryByText(/Add Person/));

    await user.click(screen.getByText(/^Select$/));
    await user.click(screen.getAllByRole('combobox')[0]);
    await user.click(screen.getByText(/^Primary Fiduciary$/));
    await user.click(screen.getByText(/Save/));
  });
});
```

```tsx
describe('BeneficiarySelectScreen', () => {
  it('should allow user to add, edit, and remove beneficiaries', async () => {
    const { user } = setup();

    await screen.findByText(/Mary Smith/);
    await user.click(screen.getByText(/\+ Add Beneficiary/i));
    expect(screen.getByText(/Recipient \*/)).toBeInTheDocument();
    // ...

    await user.click(screen.getAllByText(/Select.../)[0]);
    // Select child (Bob Smith) as recipient
    await user.click(screen.getByText(/Bob Smith/));

    await user.clear(screen.getByLabelText(/Percentage \*/));
    await user.type(screen.getByLabelText(/Percentage \*/), '12');
    expect(screen.getByLabelText(/Percentage \*/)).toHaveVlaue('12%');

    await user.click(screen.getByLabelText(/Outright$/));

    // Because Bob Smith is a child, this should not show
    expect(
      screen.queryByText(/Is this individual your descendant?/)
    ).not.toBeInTheDocument();

    await user.click(screen.getByLabelText(/Shares get divided/));

    await user.click(screen.getByText(/Save/)); // Closes Add Beneficiary modal
    await waitFor(() => {
      expect(screen.getByText(/12%/)).toBeInTheDocument();
    });

    await user.click(screen.getByText(/Continue/));
    expect(screen.getByText(/Percentage must equal 100%/)).toBeInTheDocument();

    // ...
  });

  it('should allow user to add new person when adding beneficiaries', async () => {
    const { user } = setup(
      [],
      [
        mockCreatePerson({
          birthDate: new Date('1982-07-01T00:00:00.000Z').toISOString()
        }),
        mockGetDocumentPackageRefetchWithPerson
      ]
    );

    await screen.findByText(/\+ Add Beneficiary/i);
    await user.click(screen.getByText(/\+ Add Beneficiary/i));
    await user.click(screen.getAllByText(/Select.../)[0]);
    await user.click(screen.getByText(/\+ Add New Person/));
    await user.type(screen.getByLabelText(/Full Legal Name\*/), 'Text Person');
    await user.type(
      screen.getByPlaceholderText(/MM\/DD\/YYYY/),
      '{arrowLeft}{arrowLeft}07011982'
    );

    await waitFor(() => {
      screen.getAllByRole('dialog').forEach(async modal => {
        if (within(modal).queryByRext(/Add Person/)) {
          await user.click(within(modal).getByText(/Save/));
        }
      });
    });

    await waitForElementToBeRemoved(() => screen.queryByText(/Add Person/));

    await user.clear(screen.getByLabelText(/Percentage \*/));
    await user.type(screen.getByLabelText(/Percentage \*/), '100');
    await user.click(screen.getByLabelText(/In a Trust$/));
    await user.click(screen.getByLabelText(/Yes$/));
    await user.click(screen.getByLabelText(/Shares go to the descendants/));
    await user.click(screen.getByText(/Save/));

    expect(screen.getByText(/Test Person/)).toBeInTheDocument();
    expect(screen.getByText(/Descendant/)).toBeInTheDocument();
    expect(screen.getByText(/100%/)).toBeInTheDocument();

    await user.click(screen.getByText(/Continue/));
    expect(mockSubmitNextQuestion).toBeCalledTimes(1);

    const parsedArg = JSON.parse(mockSubmitNextQuestion.mock.calls[0][0]);
    // Verify that the args are correct
    expect(parsedArg[0].id).toBe('100');
    expect(parsedArg[0].isDescendant).toBe(true);
    expect(parsedArg[0].percentage).toBe(100);
    expect(parsedArg[0].receiveOutright).toBe(false);
    expect(parsedArg[0].sharesToDescendants).toBe(true);
  });
});
```
