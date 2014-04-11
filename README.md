table-mapper
============

**TableMapper** is a class that I wrote to ease the consumption of complex HTML tables.

**Example**:

```html
<table>
    <tbody>
        <tr>
            <td colspan="3">Row 0 : Cell 0</td>
            <td rowspan="2">Row 0 : Cell 1</td>
        </tr>
        <tr>
            <td>Row 1 : Cell 0</td>
            <td>Row 1 : Cell 1</td>
            <td>Row 1 : Cell 2</td>
        </tr>
    </tbody>
</table>
```

Looks something like this:

<table>
    <tbody>
        <tr>
            <td colspan="3">Row 0 : Cell 0</td>
            <td rowspan="2">Row 0 : Cell 1</td>
        </tr>
        <tr>
            <td>Row 1 : Cell 0</td>
            <td>Row 1 : Cell 1</td>
            <td>Row 1 : Cell 2</td>
        </tr>
    </tbody>
</table>

That can be tricky to consume, however.  So this class creates a clone of the table element you pass in and creates an in-memory version that looks like this:

```html
<table>
    <tbody>
        <tr>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 1</td>
        </tr>
        <tr>
            <td>Row 1 : Cell 0</td>
            <td>Row 1 : Cell 1</td>
            <td>Row 1 : Cell 2</td>
            <td>Row 0 : Cell 1</td>
        </tr>
    </tbody>
</table>
```

Which looks like this:

<table>
    <tbody>
        <tr>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 0</td>
            <td>Row 0 : Cell 1</td>
        </tr>
        <tr>
            <td>Row 1 : Cell 0</td>
            <td>Row 1 : Cell 1</td>
            <td>Row 1 : Cell 2</td>
            <td>Row 0 : Cell 1</td>
        </tr>
    </tbody>
</table>

**Usage**

```php
$tableHTML = <<<HTML
<h1>Original Table</h1>

<table>
    <tbody>
        <tr>
            <td colspan="3">Row 0 : Cell 0</td>
            <td rowspan="2">Row 0 : Cell 1</td>
        </tr>
        <tr>
            <td>Row 1 : Cell 0</td>
            <td>Row 1 : Cell 1</td>
            <td>Row 1 : Cell 2</td>
        </tr>
    </tbody>
</table>
HTML;

$dom = new \DOMDocument;
$dom->loadHTML($tableHTML);

$tableMapper = new \DCarbone\TableMapper($dom->getElementsByTagName('table')->item(0));
$tableMapper->createMap();

$dom->appendChild($dom->createElement('h1', 'Parsed Table'));

$newTable = $dom->createElement('table');
$dom->appendChild($newTable);

foreach($tableMapper->getRowCellMap() as $groupi=>$groupDef)
{
    foreach($groupDef as $rowi=>$cellMap)
    {
        $newTr = $dom->createElement('tr');
        $newTable->appendChild($newTr);

        foreach($cellMap as $cellNum)
        {
            $cell = $tableMapper->getCell($groupi, $rowi, $cellNum);
            $newTr->appendChild($dom->createElement('td', $cell->nodeValue));
        }
    }
}

echo $dom->saveHTML();
```

This is a very simple class, overall.  A few things to note:

- TableMapper does NOT mutate the DOMElement you pass in, it clones and imports it into a new DOM before doing anything.
- TableMapper does not fix invalid HTML
- TableMapper does not do your laundry.