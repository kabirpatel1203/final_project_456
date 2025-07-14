# final_project_456


#![no_std]
#![no_main]

extern crate r_efi;

use core::ffi::c_void;
use r_efi::efi;
use core::ptr::null_mut;

// Define entry point symbol name (matches PE/COFF symbol)
#[no_mangle]
pub extern "C" fn efi_main(
    image_handle: efi::Handle,
    system_table: *mut efi::SystemTable
) -> efi::Status {
    unsafe {
        if system_table.is_null() {
            return efi::Status::LOAD_ERROR;
        }

        let con_out = (*system_table).con_out;
        if con_out.is_null() || (*con_out).output_string.is_none() {
            return efi::Status::LOAD_ERROR;
        }

        let message = wide!("Hello from r-efi DXE driver!\r\n");

        ((*con_out).output_string.unwrap())(con_out, message.as_ptr());
    }

    efi::Status::SUCCESS
}



macro_rules! wide {
    ($lit:literal) => {{
        const WIDE_CSTR: &[u16] = &{
            const S: &str = $lit;
            const LEN: usize = S.len();
            let mut buf = [0u16; LEN + 1];
            let bytes = S.as_bytes();
            let mut i = 0;
            while i < LEN {
                buf[i] = bytes[i] as u16;
                i += 1;
            }
            buf
        };
        WIDE_CSTR
    }};
}
