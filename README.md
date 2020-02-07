# XML-Store
Store Data In XML File


using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;
using System.Xml.Linq;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using RifeAPI.Models;

namespace RifeAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : ControllerBase
    {
        private IHostingEnvironment _hostingEnvironment;
        private string _filename = "";
        public ValuesController(IHostingEnvironment hostingEnvironment)
        {
            this._hostingEnvironment = hostingEnvironment;
            _filename = _hostingEnvironment.WebRootPath;
        }


        [HttpGet("Collect")]
        public string Collect(string key, string macid = "", Enums info = Enums.All)
        {
            try
            {
                var dt = DateTime.Now.AddMinutes(-2);
                string filename = Path.Combine(_filename, "App_Data/RifePrint.xml");

                XElement selected;
                int Counter = 0;
                do
                {
                    //var macid = "";
                    XDocument doc = XDocument.Load(filename);
                    if (macid == "")
                        selected = doc.Root.Elements("data").Where(b => (string)b.Attribute("key") == key && Convert.ToDateTime((string)b.Attribute("CreatedOn")) >= dt).FirstOrDefault();
                    else
                        selected = doc.Root.Elements("data").Where(b => (string)b.Attribute("key") == key && (string)b.Attribute("MachineID") == macid && Convert.ToDateTime((string)b.Attribute("CreatedOn")) >= dt).SingleOrDefault();

                    Counter++;
                }
                while (selected == null && Counter < 10000);
                if (info == Enums.All)
                    return selected != null ? selected.Value + '#' + selected.Attribute("MachineID").Value : string.Empty;
                else if (info == Enums.Printers)
                    return selected != null ? selected.Value : string.Empty;
                else
                    return selected != null ? selected.Attribute("MachineID").Value : string.Empty;
            }
            catch (Exception ex)
            {

                throw ex;
            }
        }

        [HttpGet("SaveData")]
        public string SaveData(string key, string value, string macid)
        {
            var dt = DateTime.Now;
            try
            {

                string filename = Path.Combine(_filename, "App_Data/RifePrint.xml");
                XDocument doc = XDocument.Load(filename);

                XElement root = doc.Root.Elements("data").Where(b => (string)b.Attribute("key") == key && (string)b.Attribute("MachineID") == macid).FirstOrDefault();


                if (root != null)
                {
                    root.SetAttributeValue("CreatedOn", dt.ToString());
                    if (value != null)
                    {
                        root.Value = value;
                    }
                    //doc.Root.Elements("data").Where(b => (string)b.Attribute("key") == key && (string)b.Attribute("MachineID") == macid ).FirstOrDefault().Value = value;
                    //doc.Root.Element("data").Attribute("CreatedOn").Value = dt.ToString();
                    doc.Save(filename);
                }
                else
                {
                    XElement objNewEle = new XElement("data");
                    //Attribute1
                    XAttribute objNewAttr = new XAttribute("key", key);
                    objNewEle.Add(objNewAttr);
                    //Attribute2
                    XAttribute objNewAttr2 = new XAttribute("CreatedOn", DateTime.Now.ToString());
                    objNewEle.Add(objNewAttr2);
                    //Attribute3
                    XAttribute objNewAttr3 = new XAttribute("MachineID", macid);
                    objNewEle.Add(objNewAttr3);
                    // XAttribute objNewAttr1 = new XAttribute("value", value1);

                    if (value != null)
                    {
                        objNewEle.Value = value;
                    }
                    doc.Root.Add(objNewEle);
                   
                }
                doc.Save(filename);
                return string.Empty;
            }
            catch (Exception ex)
            {
                throw ex;
            }
        }
    }
}

Note :-- After this process for store output make the RifePrint.xml File in wwwroot Folder

OutPut:-
<?xml version="1.0" encoding="utf-8"?>
<root>
  <data key="http://localhost:2928" CreatedOn="17-01-2020 01:06:51 PM" MachineID="920F-C74A-9A97-FE5E-1C90-B6ED-1166-7D25">Snagit 12</data>
</root>
