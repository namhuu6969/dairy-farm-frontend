# Frontend Rules

- Using general components (like button, table, modal,...) from folder components
- Split components in a big component to manage folder code
- Using format in utils to format date or format enum from api

# 📁 Folder Structure

- ├─public/
- │ ├─i18n # Language transfer data of i18n
- │ ├─firebase-messaging-sw.js # Config firebase for push notification
- ├─src/
- │ ├─common # Static value like primary color,...
- │ ├──components # Reusable components and ui
- │ ├──config # Config axios, i18n and other (if have)
- │ │ ├─axios # Config axios with interceptors, handle refresh token call api
- │ │ ├─i18n # Config i18n for multi-languages
- │ │ └─routes # Routes of websites
- │ ├─core # Layout and Store
- │ │ ├─layout # General layout of websites includes header (username and dropdown action includes profile, logout ), sidebar (items for navigate to another pages) and content (for render content of pages)
- │ │ └─store # State management config using redux toolkit + redux persist and contain slice too
- │ ├─hooks # Custom hook
- │ ├─pages # Contain pages ui of websites
- │ ├─service # Contain api and data for service
- │ │ ├─api # Path of api (using with swr custom hook for call api action)
- │ │ └─data # Static data or enum such as Status, Type,... for options in select (For example)
- │ ├─model # Typescript interface
- │ ├─styles # General and reusable style by using Scss
- │ └─utils # Reusable function such as format, validate or status render

# 🔁 Flow Call API Tutorial

- [x] Define Path of API in service/api
      **_Example of define path_**

```js
export const COW_PATH = {
  COWS: 'cows',
  COW_DETAIL: (id: string) => `cows/${id}`,
  COW_UPDATE: (id: string) => `cows/${id}`,
  COW_CREATE: 'cows/create',
};
```

- [x] Method Call
-       ├─GET: Call with data, mutate (Mutate will refetch api get when do action method) with method 'GET'
  **_Example of call api GET method_**

```js
const {
    data,
    error,
    isLoading,
    mutate: mutateCows,
  } = useFetcher<Cow[]>(COW_PATH.COWS, 'GET');
```

-      └─Action Method (POST, PUT, DELETE): Call with trigger for do action of this api with method action
  **_Example of call action api method_**

```js
const { trigger, isLoading } = useFetcher(COW_PATH.COW_CREATE_SINGLE, 'POST');
const handleFinish = async (values: any) => {
  if (currentStep < steps.length - 1) {
      const generalData = {
        cowStatus: values.cowStatus,
        dateOfBirth: dayjs(values.dateOfBirth).format('YYYY-MM-DD'),
        dateOfEnter: dayjs(values.dateOfEnter).format('YYYY-MM-DD'),
        cowOrigin: values.cowOrigin,
        gender: values.gender,
        cowTypeId: parseInt(values.cowTypeId, 10),
        description: stripHtml(values.description),
      };
      setGeneralInfo(generalData);
      form.resetFields(['description']);
      setCurrentStep((prev) => prev + 1);
    } else {
      try {
        const healthData = {
          status: values.status,
          description: stripHtml(values.description),
          size: parseFloat(values.size) || 0,
          bodyLength: parseFloat(values.bodyLength) || 0,
          bodyTemperature: parseFloat(values.bodyTemperature) || 0,
          chestCircumference: parseFloat(values.chestCircumference) || 0,
          heartRate: parseFloat(values.heartRate) || 0,
          respiratoryRate: parseFloat(values.respiratoryRate) || 0,
          ruminateActivity: parseFloat(values.ruminateActivity) || 0,
        };
        const payload = {
          cow: {
            cowStatus: generalInfo.cowStatus,
            dateOfBirth: generalInfo.dateOfBirth,
            dateOfEnter: generalInfo.dateOfEnter,
            cowOrigin: generalInfo.cowOrigin,
            gender: generalInfo.gender,
            cowTypeId: generalInfo.cowTypeId,
            description: generalInfo.description,
          },
          healthRecord: healthData,
        };
        const response = await trigger({ body: payload }); /// This is call API Post after handle form
        toast.showSuccess(response.message);
        const cowId = response.data?.cowId;
        if (cowId) {
          navigate(`/dairy/cow-management/${cowId}`);
        } else {
          navigate('/dairy/cow-management');
        }

        setCurrentStep(0);
        setGeneralInfo(null);
        handleClear();
      } catch (error: any) {
        toast.showError(error.message);
      }
    }
}
```

# 📚 Tech Stack

- React + Vite + Typescript
- Antd
- SWR
- Firebase
- Axios
- Tailwind, SCSS
- Redux Toolkit + Redux Persist
- i18n
- xyflow

# ♻️ How to use important reusable components

## TableComponent

- Define columns for table (dataIndex, key, title is required)
  - dataIndex is important, it will be match with property (such as data.name, dataIndex is name)
  - key is define for unique of column
  - title is header of column
  - render is handle render of that column element if it is complicated
  - sorter will enable the sort for column
  - filterable and filterOptions if need to filter base on select
  - filterDate will enable filter by date
- Call table with column and dataSource is required props
     **_For example_**

```jsx
const columns: Column[] = [
  {
    dataIndex: 'name',
    key: 'name',
    title: t('Name'),
    render: (element: string, data) => (
      <TextLink
        to={`/dairy/cow-management/${data.cowId}`}
        className="!text-base font-bold"
      >
        {element}
      </TextLink>
    ),
    searchText: true,
  },
  {
    dataIndex: 'dateOfBirth',
    key: 'dateOfBirth',
    title: t('Date Of Birth'),
    render: (data) => formatDateHour(data),
    sorter: (a: any, b: any) =>
      new Date(a.dateOfBirth).getTime() - new Date(b.dateOfBirth).getTime(),
    filteredDate: true,
  },
  {
    dataIndex: 'dateOfEnter',
    key: 'dateOfEnter',
    title: t('Date Of Enter'),
    render: (data) => formatDateHour(data),
    sorter: (a: any, b: any) =>
      new Date(a.dateOfEnter).getTime() - new Date(b.dateOfEnter).getTime(),
    filteredDate: true,
  },
  {
    dataIndex: 'dateOfOut',
    key: 'dateOfOut',
    title: t('Date Of Out'),
    render: (data) => (data ? formatDateHour(data) : '-'),
    sorter: (a: any, b: any) =>
      new Date(a.dateOfEnter).getTime() - new Date(b.dateOfEnter).getTime(),
    filteredDate: true,
  },
  {
    dataIndex: 'cowOrigin',
    key: 'cowOrigin',
    title: t('Origin'),
    render: (data) => getLabelByValue(data, cowOrigin()),
    filterable: true,
    filterOptions: cowOriginFiltered(),
  },
  {
    dataIndex: 'gender',
    key: 'gender',
    title: t('Gender'),
    render: (data) => (
      <div className="flex justify-center items-center">
        {data === 'male' ? (
          <IoMdMale className="text-blue-600" size={20} />
        ) : (
          <IoMdFemale className="text-pink-600" size={20} />
        )}
      </div>
    ),
    filterable: true, // Enables dropdown filter
    filterOptions: [
      { text: t('Male'), value: 'male' },
      { text: t('Female'), value: 'female' },
    ],
  },
  {
    dataIndex: 'cowType',
    key: 'cowType',
    title: t('Cow Type'),
    render: (data) => <p>{data.name}</p>,
    filterable: true,
    filterOptions: optionsCowTypes,
    objectKeyFilter: 'name',
  },
  {
    dataIndex: 'cowStatus',
    key: 'cowStatus',
    title: t('Cow Status'),
    render: (data) => getLabelByValue(data, cowStatus()),
    filterable: true,
    filterOptions: COW_STATUS_FILTER(),
  },
  {
    dataIndex: 'inPen',
    key: 'inPen',
    title: t('In Pen'),
    render: (data) =>
      data ? (
        <CheckCircleOutlined style={{ color: 'green' }} />
      ) : (
        <CloseCircleOutlined style={{ color: 'red' }} />
      ),
    filterable: true,
    filterOptions: [
      {
        text: t('In pen'),
        value: true,
      },
      {
        text: t('Not in pen'),
        value: false,
      },
    ],
  },
];
return (
  <TableComponent
    loading={isLoading}
    columns={columns}
    dataSource={data ? formatSTT(data) : []}
  />
);
```

## ModalComponent

- Open is required to handle state status of modal (open or close)
  - Modal will have 2 default action (onOk, onCancel), can be replaced by using footer props
  - Have disabledButtonOk if you don't want allow click Confirm if it's condition
- Using useModal custom hook to easy handle state of modal
     **_For example_**

```jsx
/// ModalCreateUser.tsx
const ModalCreateUser = ({ mutate, modal }: ModalCreateUserProps) => {
  const onFinish = async (values: CreateUser) => {
    try {
      const response = await trigger({ body: values });
      toast.showSuccess(response.message);
      onClose();
    } catch (error: any) {
      toast.showError(error.message);
    } finally {
      mutate();
    }
  };
  const onClose = () => {
    modal.closeModal();
    form.resetFields();
  };
  return (
    <div>
      <ButtonComponent onClick={modal.openModal} type="primary">
        {t('Create User')}
      </ButtonComponent>
      <ModalComponent
        open={modal.open}
        onOk={() => form.submit()}
        onCancel={modal.closeModal}
        title={t('Create new user')}
        loading={isLoading}
      >
       {/**Content of modal**/}
      </ModalComponent>
    </div>
  );
};
```

```jsx
/// UserManagement.tsx
const modalCreate = useModal();

return <ModalCreateUser modal={modalCreate} mutate={mutate} />;
```

# FormComponent

1. Handle form base on Form of antd, have FormItemComponent
2. Parse form (to handle data of form) and onFinish (function when submit form)
3. Must have FormItemComponent (with name: defined form element, rules: validate form field, label: title of form element), children is input component (Input, InputNumber, DatePicker, Select, ...)
4. Must be called with useForm hooks of antd

**_For example_**

```jsx
const [form] = Form.useForm();

const onFinish = async (values: CreateUser) => {
  try {
    const response = await trigger({ body: values });
    toast.showSuccess(response.message);
    onClose();
  } catch (error: any) {
    toast.showError(error.message);
  } finally {
    mutate();
  }
};
return (
  <FormComponent form={form} onFinish={onFinish}>
    <FormItemComponent name="id" hidden>
      <Input />
    </FormItemComponent>
    <FormItemComponent
      rules={[{ required: true }]}
      name="name"
      label={<LabelForm>{t('Name')}:</LabelForm>}
    >
      <Input />
    </FormItemComponent>
    <FormItemComponent
      rules={[{ required: true }]}
      name="email"
      label={<LabelForm>{t('Email')}:</LabelForm>}
    >
      <Input />
    </FormItemComponent>

    <FormItemComponent
      rules={[{ required: true }]}
      name="roleId"
      label={<LabelForm>{t('role')}:</LabelForm>}
    >
      <Select options={role()} />
    </FormItemComponent>
)
```
