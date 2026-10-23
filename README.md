<?php
include_once(realpath(dirname(__FILE__, 2) . '/php/session.php'));
include_once(realpath(dirname(__FILE__, 2) . '/php/path.php'));
include_once(ROOT_PATH . '/php/config.php');

$copy_id = (int) ($_GET['copy_id'] ?? 0);
$book_id = (int) ($_GET['book_id'] ?? 0);
$user_id = (int) $_SESSION['user_id'];

$member = $db_connection->query("SELECT member_id FROM Members WHERE user_id = $user_id")->fetch_assoc();
if (!$member) { die("No library card on this account."); }
$member_id = (int) $member['member_id'];

$copy = $db_connection->query("SELECT copy_id, status FROM Copies WHERE copy_id = $copy_id")->fetch_assoc();
if (!$copy || $copy['status'] !== 'in') { die("That copy is not available."); }

$due = date('Y-m-d', strtotime('+14 days'));
$db_connection->query("INSERT INTO Loans (copy_id, member_id, checkout_date, due_date) VALUES ($copy_id, $member_id, CURDATE(), '$due')");
$db_connection->query("UPDATE Copies SET status = 'out' WHERE copy_id = $copy_id");

$in = $db_connection->query("SELECT copy_id FROM Copies WHERE book_id = $book_id AND status = 'in' LIMIT 1")->fetch_assoc();
if ($in) { die("A copy is in. Check it out instead of placing a hold."); }

$db_connection->query("INSERT INTO Holds (book_id, member_id, requested_at, status) VALUES ($book_id, $member_id, NOW(), 'waiting')");
header("Location: my_loans.php");
exit;
