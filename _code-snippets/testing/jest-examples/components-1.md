# Examples - Components

```tsx
// SelectInput/Option.tsx
import { components, type OptionProps } from 'react-select';

type BaseSelectOption<T = string> = {
  __typename?: string;
  id?: string;
  label: string;
  value: T;
};

export type OptionInlineAction<T = string> = {
  icon: 'edit';
  onClick?: (option: BaseSelectOption<T>) => void;
};

export type SelectOption<T = string> = BaseSelectOption<T> & {
  action?: () => void;
  inlineActions?: OptionInlineAction<T>[];
};

const Option = <OptionValue,>(
  props: OptionProps<SelectOption<OptionValue>>
) => {
  const { children, ...restProps } = props;
  const option = props.data;

  return (
    <components.Option {...restProps}>
      1
      <OptionInnerWrapper>
        {children}
        <OptionInlineActions>
          {option.inlineActions?.map(action => (
            <button
              aria-label="Edit"
              key={action.icon}
              type="button"
              onClick={event => {
                event.preventDefault();
                event.stopPropagation();
                action.onClick?.(option);
              }}
            >
              {action.icon === 'edit' && <PencilWrite fontSize={18} />}
            </button>
          ))}
        </OptionInlineActions>
      </OptionInnerWrapper>
    </components.Option>
  );
};
```

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('SelectInput', () => {
  it('should support inline actions for options', async () => {
    const user = userEvent.setup();
    const mockOnChange = jest.fn();
    const mockInlineEditAction = jest.fn();

    render(
      <SelectInput
        name="my-select"
        options={[
          {
            label: 'RED',
            value: 'red',
            inlineAction: [{ icon: 'edit', onClick: mockInlineEditAction }]
          }
        ]}
      />
    );

    await user.click(screen.getByRole('combobox'));
    expect(screen.getByText('RED')).toBeInTheDocument();
    expect(screen.getByLabelText('Edit')).toBeInTheDocument();

    await user.click(screen.getByLabelText('Edit'));
    expect(mockInlineEditAction).toBeCalledWith(
      expect.objectContaining({
        label: 'RED',
        value: 'red'
      })
    );
    // Clicking inline action should not trigger onChange
    expect(mockOnChange).not.toBeCalled();
  });
});
```
