## Contents

[Methods should do just one thing](#methods-should-do-just-one-thing)

[Fat models, skinny controllers](#fat-models-skinny-controllers)

[Don't repeat yourself (DRY)](#dont-repeat-yourself-dry)

[Prefer to use Active record over using Query Builder and raw SQL queries](#prefer-to-use-active-record-over-using-query-builder-and-raw-sql-queries)

[Do not execute queries in View](#do-not-execute-queries-in-view)

[Follow Codeigniter naming conventions](#follow-codeigniter-naming-conventions)

### **Methods should do just one thing**

A function should do just one thing and do it well.

Bad:

```php
public function update()
{
    //updating data
    $data = array(
        'CHR_PROBLEM_TYPE'  => $this->input->post('CHR_PROBLEM_TYPE'),
        'CHR_PROBLEM_TITLE' => $this->input->post('CHR_PROBLEM_TITLE'),
        'CHR_PROBLEM_DESC'  => $this->input->post('CHR_PROBLEM_DESC'),
        'CHR_MODIFIED_BY'   => $this->session_array['USERNAME'],
        'DAT_MODIFIED_AT'   => $this->datetime
    );

    $id = array(
        'INT_ID' => $this->input->post('INT_ID'),
    );

    $this->Ticket->update($data, $id);

    //sending email
    $this->load->library('email', array('mailtype' => 'html'));
    $this->email->from("helpdesk@aii.co.id", "AII");
    $this->email->to('user@aii.co.id');
    $this->email->subject('Ticket');
    $this->email->message('Update ticket');
    $this->email->send();

    redirect($this->back_to_index);
}
```

Good:

```php
public function update()
{
    $data = array(
        'CHR_PROBLEM_TYPE'  => $this->input->post('CHR_PROBLEM_TYPE'),
        'CHR_PROBLEM_TITLE' => $this->input->post('CHR_PROBLEM_TITLE'),
        'CHR_PROBLEM_DESC'  => $this->input->post('CHR_PROBLEM_DESC'),
        'CHR_MODIFIED_BY'   => $this->session_array['USERNAME'],
        'DAT_MODIFIED_AT'   => $this->datetime
    );

    $id = array(
        'INT_ID' => $this->input->post('INT_ID'),
    );

    $this->Ticket->update($data, $id);

    redirect($this->back_to_index);
}
```

[🔝 Back to contents](#contents)

### **Methods should do just one thing**

A function should do just one thing and do it well.

Bad:

```php
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

Good:

```php
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

[🔝 Back to contents](#contents)

### **Fat models, skinny controllers**

Put all DB related logic into Active record models.

Bad:

```php
public function index()
{
    $this->db->select("*", FALSE);
    $this->db->from($this->client);

    if ($where)
        $this->db->where($where);
    $clients =  $this->db->get()->result();

    $data['clients'] = $clients;
    return view('index', $data);
}
```

Good:

```php
public function index()
{
    $data['clients'] = $this->client->get(array('INT_FLG_DEL' => 0))->result();
    return view('client', $data);
}

class Client extends CI_Model
{
    public function get($where = 0)
    {
        $this->db->select("*", FALSE);
        $this->db->from($this->client);

        if ($where)
            $this->db->where($where);
        return $this->db->get();
    }
}
```

[🔝 Back to contents](#contents)

### **Don't repeat yourself (DRY)**

Reuse code when you can. Avoid duplication.

Bad:

```php
public function getClientNonActive()
{
    return $this->db->query("SELECT * FROM TM_CLIENT WHERE INT_FLG_ACTIVE = 1");
}

public function getClientActive()
{
    return $this->db->query("SELECT * FROM TM_CLIENT WHERE INT_FLG_ACTIVE = 0");
}
```

Good:

```php
public function getClientNonActive()
{
    $client = $this->client->get(array('INT_FLG_ACTIVE' => 0))->result();
    ....
}

public function getClientActive()
{
    $client = $this->client->get(array('INT_FLG_ACTIVE' => 1))->result();
    ....
}

class Client extends CI_Model
{
    ....
    public function get($where = 0)
    {
        $this->db->select("*", FALSE);
        $this->db->from($this->client);

        if ($where)
            $this->db->where($where);
        return $this->db->get();
    }
}
```

[🔝 Back to contents](#contents)

### **Prefer to use Active record over using Query Builder and raw SQL queries.**

Active record allows you to write readable and maintainable code.

Bad:

```sql
return $this->db->query("SELECT * FROM TM_CLIENT");
```

Good:

```php
$this->db->select("*", FALSE);
$this->db->from($this->client);
return $this->db->get();
```

[🔝 Back to contents](#contents)

### **Do not execute queries in view**

Bad:

```php
$users = $this->User->get()->result();

foreach ($users as $user) {
    echo $user->CHR_NPK;
}
```

Good:

```php
foreach ($users as $user) {
    echo $user->CHR_NPK;
}
```

[🔝 Back to contents](#contents)

### **Follow Codeigniter naming conventions**

Follow [PSR standards](https://www.php-fig.org/psr/psr-12/).

Also, follow naming conventions accepted by Laravel community:

| What                                                                  | How                                                                       | Good                                    | Bad                                                             |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------- | --------------------------------------------------------------- |
| Controller                                                            | singular                                                                  | ArticleController                       | ~~ArticlesController~~                                          |
| Model                                                                 | singular                                                                  | User                                    | ~~Users~~                                                       |
| View                                                                  | kebab-case                                                                | show-user.php                           | ~~showFiltered.php, show_filtered.blade.php~~                   |
| Method                                                                | camelCase                                                                 | getAll                                  | ~~get_all~~                                                     |
| Variable                                                              | camelCase                                                                 | $articlesWithAuthor                     | ~~$articles_with_author~~                                       |
| Collection                                                            | descriptive, plural                                                       | $activeUsers = User->get()->result()    | ~~$active, $data~~                                              |
| Object                                                                | descriptive, singular                                                     | $activeUser = User->get()->row()        | ~~$users, $obj~~                                                |


[🔝 Back to contents](#contents)
