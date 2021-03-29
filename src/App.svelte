<script>
  import { PDFDocument } from 'pdf-lib'

	let fileName = "Please Upload a PDF"

  let preview;

  let pdfDoc;

  const getFieldValue = {
    PDFCheckBox: f=> f.isChecked(),
    PDFTextField: f => f.getText(),
    PDFDropdown: f => f.getSelected(),
    PDFRadioGroup: f => f.getSelected(),
    PDFOptionList: f => f.getSelected(),
  }

  let pdfFields = [];

  function handleFile(event){
    const tmp = []
    const file = event.target.files[0]
    fileName = file.name
    const reader = new FileReader()

    reader.onload = async function(){
      console.log("Loading File")
      const typedarray = new Uint8Array(this.result)
      pdfDoc = await PDFDocument.load(typedarray).catch(err => console.error(err))
      const form = await pdfDoc.getForm()
      const fields = await form.getFields()
      fields.forEach(field => {
        const type = field.constructor.name
        const name = field.getName()
        const value = type in getFieldValue ? getFieldValue[type](field) : "NOT SUPPORTED"
        console.log(`${type}: ${name} - ${value}`)
        tmp.push({type, name, value})
      })
      pdfFields = tmp

      //await PDFObject.embed(await pdfDoc.saveAsBase64({ dataUri: true }), preview, {height: "80vh"});
      preview = await pdfDoc.saveAsBase64({dataUri:true})
    }

    reader.readAsArrayBuffer(file);
  }
</script>

<section class="section">

  <h1 class="title">
	  PDF Test
  </h1>
  <div class="columns">
    <div class="column">
      <div class="file has-name">
        <label class="file-label">
          <input class="file-input" type="file" name="resume" on:change={handleFile}>
          <span class="file-cta">
            <span class="file-icon">
              <i class="fas fa-upload"></i>
            </span>
            <span class="file-label">
              Choose a file…
            </span>
          </span>
          <span class="file-name">
            {fileName}
          </span>
        </label>
      </div>
    
      {#if pdfFields.length}
      <table class="table is-bordered is-hoverable is-striped">
        <thead>
          <tr>
            <th>Reference</th>
            <th>Reference</th>
            <th>Value</th>
          </tr>
        </thead>
        <tbody>
          {#each pdfFields as f}
            <tr>
              <td>{f.name}</td>
              <td>{f.type}</td>
              <td>
                {f.value}
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
      {/if}
    </div>
    <div class="column">
      Preview
      {#if preview}
      <object title="PDF Preview" data={preview} type="application/pdf" style="overflow: auto; width: 100%; height: 80vh; pointer-events: none;"/>
      {/if}
    </div>
  </div>
</section>