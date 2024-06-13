using BBMS_BL;
using BBMS_Entities;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace BBMS_Core_Web_API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BloodTransferController : ControllerBase
    {

        //PUT: /Transfer
        [Route("Transfer")]
        [HttpPut]
        //[ResponseType(typeof(BloodInventory))]
        public BloodBankManagementSystemHttpStatusCode PutBloodInventoryBL(string bloodGroup, int requirement)
        {
            //write validation for blood group and requirement in bl
            BloodTransferBL inventory = new BloodTransferBL();

            return inventory.UpdateBloodInventoryBL(bloodGroup, requirement);
        }


        // GET: /GetIfRequiredBottlesExist
        [Route("GetIfRequiredBottlesExist")]
        [HttpGet]
        // [ResponseType(typeof(BloodInventory))]

        public async Task<ActionResult> GetIfRequiredBottlesExist(int bloodId, int requirement)
        {
            BloodTransferBL inventory = new BloodTransferBL();
            if (inventory.GetIfRequiredBottlesExistBL(bloodId, requirement) >= requirement)
            {
                return Ok("Bottles exist for transfer");
            }
            return NotFound();

        }

    }
}
